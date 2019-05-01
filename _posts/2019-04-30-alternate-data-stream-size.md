---
layout: post
title:  "Putting the ADS in Madness"
date:   2019-04-30 17:38:00 -0500
categories: windows ntfs
author: Colin Atkinson
---

Alternate Data Streams are a lesser known bit of NTFS weirdness. You can think of them as sort of like xattrs on Linux, except they are treated (kinda) exactly like files instead of as a special thing. Or as a macOS resource fork, compatibility with which was the original reason they were created.

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
        # TODO: Find out what that's being translated from
        print(path, ct)


if __name__ == "__main__":
    run_test("a.txt", gen_sha256)
    run_test("b.txt", gen_sha128)

    # TODO: Get output to show
```

# TODO: Finish

## Further Resources

* [NTFS Cheat Sheet](http://www.writeblocked.org/resources/NTFS_CHEAT_SHEETS.pdf)--a quick visual guide to the structures of the MFT.
* [Wikipedia: NTFS](https://en.wikipedia.org/wiki/NTFS#Alternate_data_streams_(ADS))
* [NTFS Attributes; Non Resident and No Named Attributes](http://sabercomlogica.com/en/ebook/ntfs-non-resident-and-no-named-attributes/)--Detailed technical information on the structure of MFT attributes.
* [Introduction to ADS](https://hshrzd.wordpress.com/2016/03/19/introduction-to-ads-alternate-data-streams/)--a high-level overview of the history of and how to interact with named streams.
* [Alternate Data Streams: Out of the Shadows and into the Light](https://www.giac.org/paper/gcwn/230/alternate-data-streams-shadows-light/104234)--a fairly in-depth overview of `$DATA` and the MFT. I also like it because the title could also work for a really cheesy sci-fi or fantasy novel.
