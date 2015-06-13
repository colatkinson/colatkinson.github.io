---
layout: post
title: Example content
---


As a part of my school's annual homecoming tradition, each class decorates one hallway to follow a certain theme. This year, we were the Senior Spartans and had a Greek theme. I had been looking for an excuse to use my [Arduino](http://www.arduino.cc/) for something, and so I figured I'd pitch in with some embedded programming.

## The Project
What I decided on was a [fortune teller machine](http://en.wikipedia.org/wiki/Fortune_teller_machine) themed as a Greek oracle. The connection to our theme was tangential at best, but hey, when do teachers take points off for adding electronics? I had a low-powered speaker that could be driven by the Arduino's PWM in much the same way as a piezo buzzer. Essentially, a user would press a button, and a pre-generated audio sample would play declaring the class of 2015 as superior. Ultimately, I chose this because I had almost all of the materials necessary to implement this.

## Materials
* Arduino Uno or compatible board
* A breadboard
* A bunch of jumper and alligator cables
* A non-powered speaker. Mine was 8 &#8486; and 0.25 W, but that's just what I had lying around.
* A talking, hanging Halloween decoration with LED eyes. I can't find the model I got online, but basically this is just the shell of the device.

## Design
I set up the system using a breadboard. There were two distinct circuits: one for the button, and one for the LEDs and the speaker. The LEDs and the speaker were laid out in parallel so that the LEDs would flicker with the voice. This wasn't necessary, but it looks pretty cool.

