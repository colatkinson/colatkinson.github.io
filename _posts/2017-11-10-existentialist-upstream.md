---
layout: post
title:  "Forks: A Solution to Your Existential Nightmare?"
date:   2017-11-10 18:16:00 -0500
categories: foss java
author: Colin Atkinson
---

Sometimes in life, things just don't work the way you want them to. Your car's battery dies in the dead of winter. Your cat knocks your coffee over onto your laptop. You accidentally fulfill a prophecy by killing your father and knocking up your mother.

These are all things that you have no control over. The universe's eternal chaos cancels your best-laid plans like Fox cancelled Firefly&mdash;swiftly, quietly, and with no regard for your feelings on the matter. But occasionally, you can actually fix your problems, without, like, actually going to a therapist.

That's one of the many benefits of open source. Unlike everything else in life, you actually have control over the precise functionality of your software. If something is broken, you fix it or deal with it.

One of the libraries LMSGrabber is using in our project, [Conductor](https://github.com/conductor-framework/conductor), is a great fit for our use case, but with one notable flaw.

It relies on [Selenium](http://www.seleniumhq.org/), which in turn relies on binaries called *webdrivers*. These provide a unified interface so that Selenium can interact automatically with multiple web browsers, like Chrome, Firefox, Edge, and PhantomJS.

Conductor is a relatively immature project. This isn't an insult, by any means. The overwhelming majority of the work is done by a single maintainer (shouts out [ddavidson](https://github.com/ddavison)). He's been steadily grinding away on it since 2014. Given this, it's remarkably stable and usable.

But it still has some rough edges.

Previously, it expected the appropriate webdrivers to be stored in the directory from which the project was executed, usually the project root. This presents a few problems:

1. These drivers are not packed into JAR executables, making it exceedingly difficult to distribute the built project.
2. Since it expects very specific file names, it does not integrate easily with [the webdriver Maven plugin](https://github.com/webdriverextensions/webdriverextensions-maven-plugin). Our temporary workaround involved some remarkably hacky Bash scripting.

So you know what I did? I marched boldly into that repo, my resolve strengthened by my fifth cup of coffee for the day, and I [submitted an issue](https://github.com/conductor-framework/conductor/issues/31). And you know what that damn maintainer did? He [responded quickly and politely, encouraging me to submit a pull request](https://github.com/conductor-framework/conductor/issues/31#issuecomment-339734133). Can you believe this jerk?

I still felt a pressure to perform. So I quickly forked the repo, and built in the functionality I wanted. One [merged pull request](https://github.com/conductor-framework/conductor/pull/32) later, and **BAM**! I'd made the open source community slightly better, while making my work on LMSGrabber easier. Plus, it gave me something to do during a particularly dry lecture.

The moral of the story is, if you see a way to make a component you're using better, then go for it. The maintainer may never have considered your use case. And 99% of the time, as long as you're respectful and clearly explain what changes you would like to make, they'll be happy to accept your contributions.

So make the world a better place. Seize control of your fate. Contribute to upstream.
