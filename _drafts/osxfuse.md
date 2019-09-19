---
layout: post
title: "FUSE for macOS is no longer open source"
description: "And its maintainer isn't being too upfront about it."
categories: macos fuse
author: Colin Atkinson
---

FUSE for macOS (or, the kernel extension formerly known as osxfuse) is a
project dating back to 2011. It in turn is based on even older projects, such
as MacFUSE, the Linux FUSE module, and even some code open sourced by Apple.

Until 2017, the project was open source, released under an amalgamation of the
fairly permissive licenses of its various ancestors. But its maintainer,
[Benjamin Fleischer](https://github.com/bfleischer), decided to change that.
While several releases have been published since then, only the binary blobs
are provided, leaving Fleischer as the only person with access to the complete
source.

We'll get into why, and how, this happened in a bit. But first, some
background.

# FUSE: What is it good for?

FUSE stands for **F**ilesystem in **Use**rspace, and the name pretty much tells
the whole story. It's a Linux kernel module that allows developers to implement
a filesystem outside of the kernel, just like a normal application. This makes
essentially the entire development process easier, since you have access to all
of the normal libraries and utilities you use in desktop development.

Some notable FUSEs are Google Drive's File Stream, sshfs, Keybase's KBFS, and
ntfs-3g. The point is, this is a piece of technology that has been used and
abused endlessly, and that a ton of projects depend on.

# But what about macOS?

As I mentioned earlier, since shortly after FUSE was created, people have wanted
their filesystems to run on other operating systems. Windows, bizarrely enough,
seems to get far more love here, with Dokan, Dokany, and WinFsp filling the
void.

Support for macOS, however, has been... limited, despite the fact that it and
Linux have far more similar sets of file operations thanks to POSIX compliance.
MacFUSE was started at Google in 2007 and, from what I can tell, was abandoned
sometime around 2009. osxfuse was then forked from it around 2011, and has been
the only solution for running your FUSE on macOS since then.

# Re-enter Google

Google has started to get serious about the whole cloud file storage thing, and
one of their more enterprise-facing products is Google Drive File Stream.
Instead of just exposing a web UI, GDFS actually exposes your files locally,
similar to Dropbox.

Unlike Dropbox, GDFS downloads files on an as-needed basis[^smart_sync]. So if I
have 10 GB of movies in my Drive, none of that 10 GB is actually used until I
open one of those files. This is great for users, especially those with lots of
rarely-accessed files. But to implement this, you need a virtual filesystem to
"trick" applications into thinking your files are there before they actually
are[^gdrive_shellext].

So how, you may ask, did Google choose to implement this functionality? Why, by
(poorly) forking osxfuse of course!

As noted in this [GitHub issue](https://github.com/osxfuse/osxfuse/issues/503),
GDFS and other osxfuse-based systems were incompatible. Essentially, it appears
that Google basically changed the name of the kernel extension, got a new kext
signing certificate from Apple, and proceeded to call it a day. Now this isn't
to cast any shade--we all have deadlines to meet, and developing a new kext from
scratch is time consuming to say the least. And I sincerely doubt that ensuring
users of sshfs could work uninterrupted was at the top of the pile during sprint
planning.

That being said, still kind of a dick move. But understandable.

I believe this was eventually fixed, since I've seen osxfuse and more recent
versions of GDFS running side by side. But the damage was already done.

# Hold up, what's a kext signing certificate?

Now this may come as a shock to some of you, but Apple *really* hates when third
party developers change just about anything about their UX. They only added an
API for developers to add badge icons and toolbar buttons to Finder in 2014--and
that was only because Dropbox just kept reverse engineering it anyway.

Deploying a kernel extension for macOS requires it be signed using a special
Kernel Extension Signing Certificate, which can only be acquired from Apple
(after you've joined their developer program and paid your $99, of course). This
is a special application process that, from what I've heard, is fairly rigorous.

Now Google, when they forked osxfuse, were able to get one of these certs. But
most software shops don't have hundreds of billions of dollars in yearly
revenue, and would presumably face more scrutiny.

Now here's the kicker: Benjamin Fleischer has one of these certificates for
osxfuse. In fact, if you google his name, the vast majority of the results are
confused users asking why they're being prompted about installing software by
ol' Benny. What can I say, the man's a celebrity.

# The perfect storm

So to quickly summarize:

1. osxfuse is used by tons of companies,
2. sometimes in stupid, selfish ways.
3. Fleischer has been the sole maintainer for years.
4. He also hasn't been paid a penny for this work.
5. He also holds a magical, aluminum unibody certificate that prevents most
   people from forking his module.
6. The current codebase is permissively licensed, so he can do with it what he
   will.

It doesn't exactly take a rocket surgeon to see where this is going.

# It gets perfecter

Apple has apparently made some pretty significant changes to kernel modules in
the upcoming macOS Catalina. And according to our subject matter expert,
[they're pretty gnarly](https://github.com/osxfuse/osxfuse/issues/595) to add
support for. Fortunately, osxfuse 3.10 has support for Catalina!

But there's a catch. If you go to download this
[release](https://github.com/osxfuse/osxfuse/releases/tag/osxfuse-3.10.0), you'll notice a bit of text in the release notes:

> The license has changed. Starting with this release, redistributions bundled
> with commercial software are not allowed without specific prior written
> permission. Please contact Benjamin Fleischer.

The last public source code release was in 2017. Now, two years later, just a
few short months before the [Fucking Catalina Release
Mixer](https://www.youtube.com/watch?v=RGB8QgwxZqw), Fleischer could finally
get his.

# So to summarize again

1. Apple does Apple things and heavily restricts third-party developers.
2. But not Google tho lol.
3. Benjamin Fleischer realizes he doesn't get paid enough for this shit.
4. He makes the repo closed source in 2017, but doesn't mention this to anyone.
5. In 2019, after making a bunch of critical changes to the code, he quietly
   announces that the licensing terms of the project are now different.

Now depending on how you look at it, that's either a power play for the gods, or
a total dick move.



[^smart_sync] Notably, Dropbox has responded to this with Project Infinite/Smart
Sync, which is quite similar at a technical level.

[^gdrive_shellext] This isn't technically 100% accurate (at least for Windows).
An earlier version of GDFS on Windows actually eschewed a traditional
filesystem, opting instead to use an Explorer shell extension to do much the
same thing.  The code is
[available](https://github.com/google/google-drive-shell-extension) if you're
into that sort of thing.
