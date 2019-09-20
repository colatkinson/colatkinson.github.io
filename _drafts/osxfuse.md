---
layout: post
title: "osxfuse is no longer open source"
description: "And what that means for the rest of us."
categories: macos fuse
author: Colin Atkinson
---

FUSE for macOS (or, the kernel extension formerly known as osxfuse) is a project
dating back to 2011. It in turn is based on even older projects, such as
MacFUSE, the Linux FUSE module, and even some code open sourced by Apple.

Until 2017, the project was open source, released under an amalgamation of the
fairly permissive licenses of its various ancestors. But its maintainer,
[Benjamin Fleischer](https://github.com/bfleischer), decided to change that.
While several releases have been published since then, only the binary blobs are
provided, leaving Fleischer as the only person with access to the complete
source. For some previous discussion, check
[here](https://github.com/osxfuse/osxfuse/issues/590),
[here](https://groups.google.com/forum/#!msg/osxfuse-group/_5PBFQ_BQB8/z1mu2H0rFAAJ),
and [here](https://github.com/osxfuse/osxfuse/issues/616).

We'll get into why, and how, this happened. But first, some background.

# FUSE: What even is that?

FUSE stands for **F**ilesystem in **Use**rspace, and the name pretty much tells
the whole story. It's a Linux kernel module that allows developers to implement
a filesystem outside of the kernel, just like a normal application. This vastly
simplifies the development process, since you have access to all of the normal
libraries and utilities you use in desktop development. It was mainlined way
back in 2005, and even then it wasn't a new concept[^hurd].

Some notable FUSEs are Google Drive's File Stream, sshfs, Keybase's KBFS, and
ntfs-3g. The point is, this is a piece of technology that has been used and
abused endlessly, and that a ton of projects depend on.

# But what about macOS?

As I mentioned earlier, since shortly after FUSE was created, people have wanted
their filesystems to run on other operating systems. Windows, bizarrely enough,
seems to get far more love here, with Dokan, Dokany, and WinFsp filling the
void.

Support for macOS, however, has been... limited, despite the fact that it and
Linux have far more similar sets of file operations thanks to POSIX. MacFUSE was
started at Google in 2007 and, from what I can tell, was abandoned sometime
around 2009. osxfuse was then forked from it around 2011, and has been the only
solution for running your FUSE on macOS since then.

# Re-enter Google

Google has started to get serious about the whole cloud file storage schtick,
and one of their more enterprise-facing products is [Google Drive File
Stream](https://support.google.com/a/answer/7491144?hl=en). Instead of just
exposing a web UI, GDFS actually exposes your files locally, similar to
Dropbox.

Unlike Dropbox, GDFS downloads files on an as-needed basis[^smart_sync]. So if I
have 10 GB of movies in my Drive, none of that 10 GB is actually used until I
open one of those files. This is great for users, especially those with lots of
rarely-accessed files. But to implement this, you need a virtual filesystem to
"trick" applications into thinking your files are there before they actually
are[^gdrive_shellext].

So how, you may ask, did Google choose to implement this functionality on
macOS? By forking osxfuse, of course!

As noted in this [GitHub issue](https://github.com/osxfuse/osxfuse/issues/503),
GDFS and other osxfuse-based systems were incompatible. Essentially, it appears
that Google basically changed the name of the kernel extension, got a new kext
signing certificate from Apple, and said, "Fuck it. Ship it." No hate here--we
all have deadlines to meet, and developing a new kext from scratch is time
consuming to say the least. I also have a sneaking suspicion that maintaining
compatibility with other osxfuse users may not have been at the top of the
sprint priority list. Or it was a legitimate accident--from what I can tell,
there's approximately one person on the planet that knows all the nooks and
crannies of the module. Regardless, it's easy to see how this slipped through
the cracks.

I believe this was eventually fixed, since I've seen osxfuse and more recent
versions of GDFS running side by side.

But the damage was already done. Plenty of other companies use osxfuse, and
while most don't break things quite as badly or as publicly, they still benefit
from Fleischer's labor and leave him to clean up any ensuing bugs and messes.

# Hold up, what's a "kext signing certificate?"

Now this may come as a shock to some of you, but Apple *really* doesn't seem to
like it when third party developers change just about anything about their UX.
They only added an API for adding badge icons to Finder in 2014--and that was
only because Dropbox just kept reverse engineering it anyway.

Deploying a kernel extension for macOS requires it be signed using a special
Kernel Extension Signing Certificate, which can only be acquired from Apple.
After you've paid your $99 to get into the standard developer program, getting
one of these certs requires going through an extra application process which,
as I understand it, is quite rigorous. And of course, there's a high
probability that you'll be denied at the end of it.

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
[they're pretty gnarly](https://github.com/osxfuse/osxfuse/issues/595) to
support. Fortunately, osxfuse 3.10 has support for Catalina!

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
3. Fleischer, having dealt with malarkey like this for close to a decade,
   realizes he doesn't get paid enough for this shit.
4. He makes the repo closed source in 2017, but doesn't mention this to anyone.
5. In 2019, after making a bunch of critical changes to the code, he quietly
   announces that the licensing terms of the project are now different.

As far as monetization strategies go, love it or hate it, you've got to give the
guy props.

# He can do that?

Uh... probably? As I mentioned, *most* of the code is under BSD-style licenses.
The command line utility to actually mount the damn thing is under the Apple
Public Source License, which has a "soft copyleft." But in theory, if no further
changes are made to this part of the code, it already meets the requirements for
source code distribution.

Now I'm not a lawyer (yada yada this shouldn't be considered legal advice), but
it seems like all of this is perfectly legal.

# Isn't there a better way?

Many open source projects take the approach of dual licensing with the GPL. This
seems to hit the requirements pretty closely. Noncommercial, open source
software can use the module for free. And companies or individuals who want to
distribute the software in a proprietary project have to pay up. As an example
in the same space, this is the strategy used by WinFsp.

It's possible that things get weird given Apple's code signing requirements, so
I'm not 100% sure if this strategy would work--that's more an issue for the
FSF's lawyers than some random programmer on the internet. But the point is,
there are probably other options than closing the source.

# So what now?

Well... nothing, really. Mr. Fleischer has come to a determination that keeping
the project open source is not a situation that benefits him. And that's
unfortunate, but at a certain point I really can't fault him.

The software which he has maintained for the past eight or so odd years has been
of enormous benefit to the software community as a whole. Instead of just being
a Linux-only thing, FUSE became a *de facto* standard for quickly spinning up a
filesystem that does cool and useful things for users. This was in part due to
his work, as well as the work of many others over the course of years.

While some of this may have come off as harsh, it's really not Fleischer with
whom we should take issue. He wouldn't be the first maintainer to have gripes
with the ways in which his labor is taken advantage of; nor is he likely to be
the last. Mongo created the SSPL due to similar issues, and Redis Labs added a
clause to its license forbidding companies from selling (parts of) its software.
These aren't directly analogous, not least because things are different in the
SaaS world. But the root cause is the same: companies often reap far more from
FOSS than they sow upstream.

All that aside: it would be super cool for osxfuse to be open sourced again, but
I don't know if I can see that happening in the near future.

The free and open source software movements have done amazing things for the
world, and while I respect the decisions made in this case, I certainly hope
this doesn't become the new normal. In the meantime, consider sending a couple
bucks to the maintainer of your favorite project. And maybe use a copyleft
license next time you release some code, so that when some random company wants
to use it, you can finagle a cut of the action.


---


### Footnotes

[^hurd]: FUSE supposedly took its inspiration from the HURD concept of [translators](https://en.wikipedia.org/wiki/GNU_Hurd#Unix_extensions). I found this interesting mostly as a historical oddity--the HURD was actually useful for something!

[^smart_sync]: Notably, Dropbox has responded to this with Project Infinite/Smart Sync, which is quite similar at a technical level.

[^gdrive_shellext]: This isn't technically 100% accurate (at least for Windows). An earlier version of GDFS on Windows actually eschewed a traditional filesystem, opting instead to use an Explorer shell extension to do much the same thing. The code is [available](https://github.com/google/google-drive-shell-extension) if you're into that sort of thing.
