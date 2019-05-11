---
layout: post
title:  "How many alternate data streams can a file have?"
date:   2019-04-30 17:38:00 -0500
categories: windows ntfs
author: Colin Atkinson
---

While I was attempting to write a <s>horrific hack</s> wonderfully elegant piece of code, I stumbled upon an undocumented feature of NTFS alternate data streams. Namely, that you can only associate so many of them with any given file. How did I hit this maximum? Just... don't worry about it, ok?

While looking into this, I found out an even weirder part: that this maximum number actually depends on the length of the streams' names.

## What the heck is an ADS?

Alternate Data Streams are a lesser known bit of NTFS weirdness. They're similar to xattrs on Linux, except you don't need a special API to read and write data to them. Just pop them open like any other file. They are also extremely similar to macOS resource forks--in fact, they were originally created for compatibility between the two systems.

The tl;dr version is as follows:

```bash
$ echo "hello" > file.txt
$ more < file.txt
hello
$ echo "yo yo yo" > file.txt:ads
$ more < file.txt:ads
yo yo yo
```

ADSes are not included in the file's size when you run `dir` or `ls`. Nor are they included in directory listings by default. They can be shown by calling `dir /r`, or with the Sysinternals `streams.exe` tool. And if you're feeling ambitious, you can even use the C `FindFirstStreamW` and `FindNextStreamW` functions to find them yourself.

ADSes are stored as part of a file's Master File Table (MFT) records. The MFT is exactly what it sounds like--the place in NTFS where all of your files' metadata (and some actual data) live. Each of the several attributes in a given MFT entry may be resident (they are stored within the entry itself) or non-resident (they are stored in a cluster somewhere, like file data). In addition to the boring stuff like file names, creation time, and DOS attributes, MFT file records house one or more `$DATA` attributes.

The `$DATA` attribute header contains much what you would expect--its own length, the length of any data it is storing internally, the cluster numbers of the data it is storing externally, and so on. It also, however, has fields for an optional name for itself. See, the "real file" is a stream just like all the ADSes--it's just unnamed. Thus, another common thing to call ADSes is "named streams."

## The weird part

As I mentioned, there seems to be a maximum number of named streams per file. This in and of itself isn't that strange: the error code returned is `ERROR_FILE_SYSTEM_LIMITATION (0x299)`, which I guess, like, fair enough. You don't want some rogue application taking up a bunch of disk space or degrading performance. But the part I just couldn't understand is this maximum seems to vary with the length of the streams' names.

### The proof

This behavior can be demonstrated fairly easily with a simple script (since they really can be used like most other files).

```python
{% include_relative scripts/named_streams.py %}
```

## Testing

Running the script will output something like

```bash
$ python named_streams.py
a.txt 1637
b.txt 2729
c.txt 4094
d.txt 5458
e.txt 6550
```

Interestingly enough, using Linux's `ntfs-3g` driver, we seem to get identical results.

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

This suggests one of two things:

 1. This is an actual technical limitation of NTFS, or
 2. `ntfs-3g` decided to maintain compatibility with the Microsoft implementation

Either way, at least we know that somewhere in some actually accessible code, there is an explanation for this weirdness.

I tried changing the cluster size and increasing the size of the MFT zone. However, neither of these seemed to change anything--the results were exactly the same. So it's not something that varies per-system.

## A clue

I decided I might as well check the system logs for any hints as to what `ntfs-3g` was doing. A quick `journalctl` later, and...

```
Too large attrlist (262208): Numerical result out of range
ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
Failed to add resident attribute: Numerical result out of range
```

Ok, this is interesting. What happens if we `grep` the source for "Too large attrlist"? This finds the following snippet in `libntfs-3g/attrib.c`.

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

What seems to be happening is that as we add more and more ADSes, the MFT entry eventually has to resort to using `$ATTRIBUTE_LIST` so that the entry can be split across multiple blocks. But it seems that `$ATTRIBUTE_LIST` has an undocumented size limit.

Since this seems to be a fairly arbitrary limitation, let's see what happens if we change this value to, say `0x80000` instead.

```bash
$ ~/named_streams.py
a.txt 3276
b.txt 5460
c.txt 8190
d.txt 10920
e.txt 13110
```

Well, that certainly seems to do it.

This limit does seem to be documented in a few places, such as [this Microsoft TechNet blog entry from 2011](https://blogs.technet.microsoft.com/mikelag/2011/02/09/how-fragmentation-on-incorrectly-formatted-ntfs-volumes-affects-exchange/):

> NTFS does have it’s limitations with the overall size of this attribute list per file and can have roughly around 1.5 million fragments. This is not an absolute maximum, but is around the area when problems can occur. The FAL size will never shrink and will continually keep growing over time. The maximum supported size of the ATTRIBUTE_LIST is 256K or 262144 [or 0x40000].

And in [The Four Stages of NTFS File Growth, Part 2](https://blogs.technet.microsoft.com/askcore/2015/03/12/the-four-stages-of-ntfs-file-growth-part-2/):

> NOTE: The diagram shows the attribute list as being smaller than the 1kb file record. And while it is true that it starts out that way, the upper limitation of the attribute list is 256kb.

## `$ATTRIBUTE_WHAT`?

In a simple file, the MFT record contains a bunch of attributes and their data (or a pointer to that data). In a more complex file, the record instead contains a list of pointers to *other* records which actually contain the attributes. This is the `$ATTRIBUTE_LIST` entry. As an aside, the `$ATTRIBUTE_LIST` itself can be made non-resident, too. NTFS supports a lot of layers of indirection.

Normally, this only becomes relevant with very large, fragmented, or sparse files (see [this post from mssql-support](https://techcommunity.microsoft.com/t5/SQL-Server-Support/Operating-System-Error-665-8211-File-System-Limitation-Not-just/ba-p/318587) for some "common" situations in which this can happen).

## Let's check it out

The obvious conclusion is that stream names take up space in `$ATTRIBUTE_LIST`--however, I couldn't find any documentation on this. Let's see...

```bash
ntfscat ntfs.img a.txt -a 0x20 > attr.bin  # 0x20 is the id of ATTRIBUTE_LIST

# Let's pick a random hash that we generated
# The chances of it occurring randomly are infinitesimal
# Note that the names are UTF-16, thus this terrible sed hack
sed 's/\x00//g' attr.bin | grep 7f2253d7e228b22a08bda1f09c516f6fead81df6536eb02fa991a34bb38d9be8
```

This should print "Binary file (standard input) matches" if all went well.

You can change the hash value to any of the hashes our script generated, and it should still match. I think it's a fair conclusion that the attribute list does actually contain stream names.

Since the size of these structures is capped not based on the number of entries, but on raw size, it suddenly makes sense that longer stream names will exhaust our 256kb more quickly.

## Conclusion

So in conclusion, it's a weird undocumented limitation that was likely added for performance reasons, and then faithfully duplicated by NTFS-3G.

I guess that the real lesson here is that if you play stupid games (writing thousands of ADSes), you win stupid prizes (things breaking). But, on the plus side I learned a lot about NTFS that I will realistically never use in my life.

## Further Resources

* [NTFS Cheat Sheet](http://www.writeblocked.org/resources/NTFS_CHEAT_SHEETS.pdf)--a quick visual guide to the structures of the MFT.
* [Wikipedia: NTFS](https://en.wikipedia.org/wiki/NTFS#Alternate_data_streams_(ADS))
* [NTFS Attributes; Non Resident and No Named Attributes](http://sabercomlogica.com/en/ebook/ntfs-non-resident-and-no-named-attributes/)--Detailed technical information on the structure of MFT attributes.
* [Introduction to ADS](https://hshrzd.wordpress.com/2016/03/19/introduction-to-ads-alternate-data-streams/)--a high-level overview of the history of and how to interact with named streams.
* [Alternate Data Streams: Out of the Shadows and into the Light](https://www.giac.org/paper/gcwn/230/alternate-data-streams-shadows-light/104234)--a fairly in-depth overview of `$DATA` and the MFT. I also like it because the title could also work for a really cheesy sci-fi or fantasy novel.
* [`$ATTRIBUTE_LIST`](https://flatcap.org/linux-ntfs/ntfs/attributes/attribute_list.html)--a fairly technical view.
* [The Four Stages of NTFS File Growth](https://blogs.technet.microsoft.com/askcore/2009/10/16/the-four-stages-of-ntfs-file-growth/)--an extremely in depth overview of how files grow, and how `$ATTRIBUTE_LIST` works.
