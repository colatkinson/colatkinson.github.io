---
layout: post
title:  "Yeah, I eat ADS"
date:   2019-04-30 17:38:00 -0500
categories: windows ntfs
author: Colin Atkinson
---

While I was attempting to write a <s>horrific hack that makes puppies cry</s> wonderfully elegant piece of code, I stumbled upon an undocumented feature of NTFS alternate data streams. Namely, that you can only associate so many of them with any given file. How did I hit this maximum? Just... don't worry about it, ok? Regardless, while looking into this, I found out an even weirder part: that this maximum number actually depends on the length of the streams' names.

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

## The Weird Part

There seems to be a maximum number of named streams per file. This in and of itself isn't that strange. The weird part is that this number seems to vary with the length of the streams' names.

### The Proof

```python
#!/usr/bin/env python3
import hashlib
import os
from typing import Callable


def gen_sha256(val: int) -> str:
    m = hashlib.sha256()
    m.update(str(val).encode("utf8"))

    return m.hexdigest()


def gen_sha128(val: int) -> str:
    orig = gen_sha256(val)

    return orig[:32]


def gen_sha64(val: int) -> str:
    orig = gen_sha256(val)

    return orig[:16]


def gen_sha32(val: int) -> str:
    orig = gen_sha256(val)

    return orig[:8]


def gen_sha24(val: int) -> str:
    orig = gen_sha256(val)

    return orig[:6]


def run_test(path: str, gen_fn: Callable[[int], str]) -> None:
    # Remove anything leftover from a previous run
    if os.path.exists(path):
        os.unlink(path)

    with open(path, "w") as f:
        f.write("F")

    ct = 0
    try:
        # Try to create up to 10000 named streams
        for ct in range(10000):
            # The form ends up being [path]:[64 or 32 bytes]
            with open(f"{path}:{gen_fn(ct)}", "w") as f:
                f.write("F")
    except OSError:
        # We expect the creation of one of the named streams to fail with
        # errno 22.
        print(path, ct)


if __name__ == "__main__":
    run_test("a.txt", gen_sha256)
    run_test("b.txt", gen_sha128)
    run_test("c.txt", gen_sha64)
    run_test("d.txt", gen_sha32)
    run_test("e.txt", gen_sha24)
```

## Testing

This will output something like

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

### Hypothesis: It's related to cluster size

It seems entirely reasonable that NTFS has a limit on how many clusters to allocate for a given file's MFT entries. This, however does not seem to be the case. Repeating the commands above, adding the `--cluster-size 8192` option to mkfs.ntfs does not seem to have an affect.

### Hypothesis: It's related to MFT size

Again, repeat the above commands and add `-z 4` (MFT takes up 50% of the disk) to `mkfs.ntfs`. However, it still appears to make no difference.

### A Clue

I decided I might as well check the system logs for any hints as to what `ntfs-3g` was doing. A `journalctl` later, and...

```
May 09 00:38:37 tenax ntfs-3g[28732]: Too large attrlist (262208): Numerical result out of range
May 09 00:38:37 tenax ntfs-3g[28732]: ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
May 09 00:38:37 tenax ntfs-3g[28732]: Failed to add resident attribute: Numerical result out of range
May 09 00:38:38 tenax ntfs-3g[28732]: Too large attrlist (262208): Numerical result out of range
May 09 00:38:38 tenax ntfs-3g[28732]: ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
May 09 00:38:38 tenax ntfs-3g[28732]: Failed to add resident attribute: Numerical result out of range
May 09 00:38:39 tenax ntfs-3g[28732]: Too large attrlist (262208): Numerical result out of range
May 09 00:38:39 tenax ntfs-3g[28732]: ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
May 09 00:38:39 tenax ntfs-3g[28732]: Failed to add resident attribute: Numerical result out of range
May 09 00:38:41 tenax ntfs-3g[28732]: Too large attrlist (262160): Numerical result out of range
May 09 00:38:41 tenax ntfs-3g[28732]: ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
May 09 00:38:41 tenax ntfs-3g[28732]: Failed to add resident attribute: Numerical result out of range
May 09 00:38:43 tenax ntfs-3g[28732]: Too large attrlist (262168): Numerical result out of range
May 09 00:38:43 tenax ntfs-3g[28732]: ntfs_non_resident_attr_expand_i: bounds check failed: Numerical result out of range
May 09 00:38:43 tenax ntfs-3g[28732]: Failed to add resident attribute: Numerical result out of range
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

What seems to be happening is that as we add more and more ADSes, the MFT entry eventually has to resort to using `$ATTRIBUTE_LIST` so that the entry can be split across multiple blocks. Something that isn't mentioned in the NTFS documentation, however, is that `$ATTRIBUTE_LIST` itself actually has a maximum size.

Since this seems to be a fairly arbitrary limitation, let's see what happens if we change this value.

# TODO: Finish

## Further Resources

* [NTFS Cheat Sheet](http://www.writeblocked.org/resources/NTFS_CHEAT_SHEETS.pdf)--a quick visual guide to the structures of the MFT.
* [Wikipedia: NTFS](https://en.wikipedia.org/wiki/NTFS#Alternate_data_streams_(ADS))
* [NTFS Attributes; Non Resident and No Named Attributes](http://sabercomlogica.com/en/ebook/ntfs-non-resident-and-no-named-attributes/)--Detailed technical information on the structure of MFT attributes.
* [Introduction to ADS](https://hshrzd.wordpress.com/2016/03/19/introduction-to-ads-alternate-data-streams/)--a high-level overview of the history of and how to interact with named streams.
* [Alternate Data Streams: Out of the Shadows and into the Light](https://www.giac.org/paper/gcwn/230/alternate-data-streams-shadows-light/104234)--a fairly in-depth overview of `$DATA` and the MFT. I also like it because the title could also work for a really cheesy sci-fi or fantasy novel.
