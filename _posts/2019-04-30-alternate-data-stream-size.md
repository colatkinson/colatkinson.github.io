---
layout: post
title:  "How many alternate data streams can a file have?"
description: "A deep dive into the data structures that make Windows work."
date:   2019-04-30 17:38:00 -0500
categories: windows ntfs
author: Colin Atkinson
image: /assets/img/attr_list_example.png
image_desc: "Sample terminal output for writing to and reading from an ADS"
---

While I was attempting to write a <s>horrific hack</s> beautiful piece of code, I stumbled upon an undocumented feature of NTFS alternate data streams. Namely, that you can only associate so many of them with any given file.

But from there I ran into something even stranger, something bizarre even by "undocumented Windows behavior" standards: that this maximum number actually depends on the length of the streams' names. Now this just didn't make sense to me, and since I didn't have anything better to do with my time, I decided to investigate.

## What the heck is an ADS?

Alternate Data Streams are a lesser known bit of NTFS weirdness. They're similar to xattrs on Linux, except you don't need a special API to read and write data to them. Just pop them open like any other file. They are also extremely similar to macOS's HFS resource forks--in fact, they were originally created for interoperability between the two filesystems.

The tl;dr version is as follows:

```bash
$ echo "hello" > file.txt
$ more < file.txt
hello
$ echo "yo yo yo" > file.txt:ads
$ more < file.txt:ads
yo yo yo
```

These alternate streams are not shown in directory listings by default, unless you run `dir` with the `/a` flag. They can also be enumerated using the `FindFirstStreamW` and `FindNextStreamW` Win32 functions if you're actually using them in application code.

Because they don't show up in obvious ways, malware often used ADSes as a way of hiding itself in older versions of Windows. The main executable would copy itself into an ADS, so even if an antivirus program attempted to srub the system, as long as that ADS and a small stub script weren't detected, the system could be reinfected. But they also have plenty of legitimate uses. The `Zone.Identifier` stream lets applications make security decisions based on where a file came from. Dropbox also uses ADSes to write unique identifiers to disk. SQL Server used to use them for integrity checking. Basically, if an application has a some bit of data and doesn't have anywhere better to put it, it ends up in an ADS.

ADSes are stored as part of a file's Master File Table (MFT) records. The MFT is where NTFS stores your files' metadata (and some of their actual data). In addition to the boring stuff like file names, creation time, and DOS attributes, MFT records for files house one or more `$DATA` attributes.

The `$DATA` attribute header contains much what you would expect--its own length, the length of any data it is storing internally, the cluster numbers of the data it is storing externally, and so on. It also, however, has fields for an optional name for itself, separate from the "traditional" file name. See, the "real" file is a stream just like all the ADSes--it's just unnamed. That's why another common thing to call ADSes is "named streams."

## The weird part

As I mentioned, there seems to be a maximum number of named streams per file. This in and of itself isn't that strange: the error code returned is `ERROR_FILE_SYSTEM_LIMITATION (0x299)`. Which I guess, like, fair enough; you don't want some rogue application taking up a bunch of disk space or degrading performance. But the part I just couldn't understand is this maximum seems to vary with the length of the streams' names.

### The proof

My general philosophy on these kinds of things is, when in doubt, whip up a Python script to reproduce the behavior. This was pretty easy to do--ADSes really do work just like files.

{% highlight python linenos %}
{% include_relative scripts/named_streams.py %}
{% endhighlight %}

## Testing

Running the script on a Windows machine gets you output like this, where the number is the number of ADSes created before the limit was reached. `a.txt` has the longest stream names, and `e.txt` the shortest.

```bash
$ python named_streams.py
a.txt 1637
b.txt 2729
c.txt 4094
d.txt 5458
e.txt 6550
```

This was enough for me to confirm the behavior, but I couldn't go much further on the Windows side. The docs were empty, and I wasn't about to start decompiling the NT kernel.

After some searching, I found out that the `ntfs-3g` FUSE actually had support for named streams, even using the same path format as Windows. So I could run my script unedited, and get some clues as to where the limitation lay.

To my surprise, the results were identical.

```bash
$ fallocate -l 4G ntfs.img
$ sudo losetup -f --show ntfs.img
/dev/loop0
$ sudo mkfs.ntfs /dev/loop0
$ sudo losetup -d /dev/loop0
$ mkdir ntfs_mnt
$ ntfs-3g -o streams_interface=windows,no_def_opts,silent ntfs.img ntfs_mnt
$ cd ntfs_mnt
$ ~/named_streams.py
a.txt 1637
b.txt 2729
c.txt 4094
d.txt 5458
e.txt 6550
$ fusemount -u ntfs_mnt
```

While I still couldn't be sure whether it was a compatibility hack or a pure technical limitation, at least I had an easier way of iterating on theories. 

I looked into the options for `mkfs.ntfs` to see what filesystem attributes I could tweak. The most compelling things to try were changing the cluster size and increasing the disk space allocated for the MFT. But neither changed the output.

I began the onerous process of looking through the `ntfs-3g` source. But I didn't really know where to begin.

## A clue

I noticed that throughout the code, errors actually got output to the system log. Maybe that could point me in the right direction. I opened up the logs and immediately found these lines.

```
Too large attrlist (262208): Numerical result out of range
ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
Failed to add resident attribute: Numerical result out of range
```

Something, somewhere was hitting a maximum of some kind, which sounded a whole lot like it was part of the named stream limit. But maybe it was unrelated--you never know. I reran my testing script, and out popped the same message. I was on to something. I searched the source for "Too large attrlist," and found the following snippet in `libntfs-3g/attrib.c`.

```c
/*
 * $ATTRIBUTE_LIST shouldn't be greater than 0x40000, otherwise 
 * Windows would crash. This is not listed in the AttrDef.
 */
if (type == AT_ATTRIBUTE_LIST && size > 0x40000) {
    errno = ERANGE;
    ntfs_log_perror("Too large attrlist (%lld)", (long long)size);
    return -1;
}
```

This was it--the source of the limit. A hardcoded size limit of `0x40000` for the file's `$ATTRIBUTE_LIST`, one of the bits of data in its MFT entry. I still wanted to be sure that this was really the root cause, though. Would changing that magic value actually change the number of named streams, or was it just a red herring?

I changed it to `0x80000`, compiled, and remounted my image. I ran the script again.

```bash
$ ~/named_streams.py
a.txt 3276
b.txt 5460
c.txt 8190
d.txt 10920
e.txt 13110
```

It actually worked! Not only did the number of streams increase, it almost exactly doubled. That `0x40000` was without a doubt the limit I was hitting. But that still didn't explain how the streams' names got dragged into this.

## But first: why 262144?

I mean, it's a pretty good number. So why *not* 262144?

I still decided to look around and see if there was anything detailing a more technical reason. This limit does seem to be documented in a few places, such as [this Microsoft TechNet blog entry from 2011](https://blogs.technet.microsoft.com/mikelag/2011/02/09/how-fragmentation-on-incorrectly-formatted-ntfs-volumes-affects-exchange/):

> NTFS does have it’s limitations with the overall size of this attribute list per file and can have roughly around 1.5 million fragments. This is not an absolute maximum, but is around the area when problems can occur. The FAL size will never shrink and will continually keep growing over time. The maximum supported size of the ATTRIBUTE_LIST is 256K or 262144 [or 0x40000].

And in [The Four Stages of NTFS File Growth, Part 2](https://blogs.technet.microsoft.com/askcore/2015/03/12/the-four-stages-of-ntfs-file-growth-part-2/):

> NOTE: The diagram shows the attribute list as being smaller than the 1kb file record. And while it is true that it starts out that way, the upper limitation of the attribute list is 256kb.

References to, but not explanations of, the limitation seem to be the extent of the information coming from inside Microsoft. Just as I was about to give up all hope, I stumbled across a reference to this value in the [Linux kernel source](https://github.com/torvalds/linux/blob/8148c17b179d8acad190551fe0fb90d8f5193990/fs/ntfs/layout.h#L959):

> The attribute list attribute value has a maximum size of 256kb. This is imposed by the Windows cache manager.

And on [another line](https://github.com/torvalds/linux/blob/8148c17b179d8acad190551fe0fb90d8f5193990/fs/ntfs/layout.h#L1852) in the same file:

> Also, each security descriptor is stored twice in the $SDS stream with a fixed offset of 0x40000 bytes (256kib, the Windows cache manager's max size) between them...

So presumably, `$ATTRIBUTE_LIST` needs to be less than 256kb so it can be cached easily. While that just pushed the magic number up a level, it did give a satisfyingly pragmatic reason for that number to be used by NTFS.

## What the heck is an `$ATTRIBUTE_LIST`?

I'd determined that the ADSes were filling up the MFT entry's `$ATTRIBUTE_LIST`. But what exactly is that anyway?

In a simple file, the MFT record contains a bunch of attributes and their data (or a pointer to that data).

In a more complex file however, the record instead contains a list of pointers to *other* records which actually contain the attributes (so-called non-resident attributes). This list of pointers to attributes is the `$ATTRIBUTE_LIST`.

Of course, the `$ATTRIBUTE_LIST` itself can be made non-resident, too. A file can have a pointer to an attribute list, which contains pointers to attributes, which contain pointers to their associated data--and I might be missing some pointers in there. It's just pointers all the way down.

<picture>
    <source srcset="/assets/gen/attr_list.dot.svg" type="image/svg+xml" />
    <img src="/assets/gen/attr_list.dot.png" alt="A diagram showing MFT entry indirection" />
</picture>

Normally, this only becomes relevant with very large, fragmented, or sparse files (see [this post from mssql-support](https://techcommunity.microsoft.com/t5/SQL-Server-Support/Operating-System-Error-665-8211-File-System-Limitation-Not-just/ba-p/318587) for some "common" situations in which this can happen). But since each named stream creates its own `$DATA` attribute, the `$ATTRIBUTE_LIST` was filling up rather quickly.

## Poking it with a stick

The tl;dr version is that `$ATTRIBUTE_LIST` contains all of the data attributes, and their name metadata. So longer names take up more raw space in the attribute list, and badabing badaboom you've got an `ERROR_FILE_SYSTEM_LIMITATION`.

While it was a nice theory, I couldn't actually find sources anywhere confirming that stream names were stored there. So it was still just that, a theory. Since I was curious about the structure of the attribute list anyway, I figured I might as well do the last bit of legwork to put all this to rest.

I first dumped the raw bytes of the `$ATTRIBUTE_LIST` using the `ntfscat` tool that comes with `ntfs-3g`. Note that 0x20 is the magic byte that identifies an MFT entry as an attribute list.

```bash
$ ntfscat ntfs.img a.txt -a 0x20 > attr.bin
```

I picked one of the SHA256 hashes generated at random. While I couldn't tell you the exact odds of a collision, it falls somewhere in the ballpark of "I quantum tunneled onto a billionaire's yacht, then joined their poker game and got dealt a royal flush." So probably good enough for these purposes.

Since grep doesn't support UTF-16, I initially used a hack; `sed` removes the upper 8 bits of each UTF16 character, leaving us with ASCII-ish text.

```bash
$ sed 's/\x00//g' attr.bin | grep 7f2253d7e228b22a08bda1f09c516f6fead81df6536eb02fa991a34bb38d9be8
Binary file (standard input) matches
```

It appears stream names are in the attribute list after all! I double-checked this manually, and yup, that sure looks like a whole lot of hashes.

![The `$ATTRIBUTE_LIST` dump open in vim](/assets/img/attribute_list_vim.png)

As a side note, I later found out that [ripgrep](https://github.com/BurntSushi/ripgrep) has good text encoding support, even when searching binary files.

```bash
$ rg -E "UTF-16LE" 7f2253d7e228b22a08bda1f09c516f6fead81df6536eb02fa991a34bb38d9be8 attr.bin
Binary file matches (found "\u{0}" byte around offset 1)
```

## Conclusion

So, to summarize this behavior, it works as follows:

 1. The large number of `$DATA` attributes causes an `$ATTRIBUTE_LIST` to be created.
 2. The full name of each stream is stored in the `$ATTRIBUTE_LIST`.
 3. `$ATTRIBUTE_LIST` is capped at 256kb, which is exhausted more quickly due to the longer stream names.
 4. When this cap is hit, the driver returns `ERROR_FILE_SYSTEM_LIMITATION`.

While the behavior is strange from an external perspective, it makes perfect sense when you actually look at the data structures being used.

Hopefully, this had some useful information about NTFS's internal structures, both the documented and the undocumented. If anyone knows of any further information related to this topic, please, let me know!

## Further Resources

* [NTFS Cheat Sheet](http://www.writeblocked.org/resources/NTFS_CHEAT_SHEETS.pdf)--a quick visual guide to the structures of the MFT.
* [Wikipedia: NTFS](https://en.wikipedia.org/wiki/NTFS#Alternate_data_streams_(ADS))
* [NTFS Attributes; Non Resident and No Named Attributes](http://sabercomlogica.com/en/ebook/ntfs-non-resident-and-no-named-attributes/)--Detailed technical information on the structure of MFT attributes.
* [Introduction to ADS](https://hshrzd.wordpress.com/2016/03/19/introduction-to-ads-alternate-data-streams/)--a high-level overview of the history of and how to interact with named streams.
* [Alternate Data Streams: Out of the Shadows and into the Light](https://www.giac.org/paper/gcwn/230/alternate-data-streams-shadows-light/104234)--a fairly in-depth overview of `$DATA` and the MFT. I also like it because the title could also work for a really cheesy sci-fi or fantasy novel.
* [`$ATTRIBUTE_LIST`](https://flatcap.org/linux-ntfs/ntfs/attributes/attribute_list.html)--a fairly technical view.
* [The Four Stages of NTFS File Growth](https://blogs.technet.microsoft.com/askcore/2009/10/16/the-four-stages-of-ntfs-file-growth/)--an extremely in depth overview of how files grow, and how `$ATTRIBUTE_LIST` works.
* [`ntfs-3g`](https://www.tuxera.com/community/open-source-ntfs-3g/)--where to download the sources of the module *without* having to touch SourceForge.
