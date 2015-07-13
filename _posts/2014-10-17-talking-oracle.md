---
layout: post
title: Making a Talking Arduino Oracle
cover: /assets/images/IMG_20141009_074612_crop.jpg
color: black-87
categories: engineering
---

A look into using an Arduino Uno and a great deal of prayer to power a Halloween decoration in the name of school spirit

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

![My pro-tier wire management](/assets/images/IMG_20141004_011721.jpg)

## Design
I set up the system using a breadboard. There were two distinct circuits: one for the button, and one for the LEDs and the speaker. The LEDs and the speaker were laid out in parallel so that the LEDs would flicker with the voice. This wasn't necessary, but it looks pretty cool.

For prototyping, I simply used discrete LEDs; however, for the final product, these would be replaced with the LEDs in the decoration.

![The pratially-finished oracle](/assets/images/IMG_20141003_232914.jpg)

## Programming
This is where the fun began. Given the limitations of both the microcontroller and the speaker, only extraordinarily simple sound could be produced. In the past I had created simple square waves with [pulse-width modulation](https://en.wikipedia.org/wiki/Pulse-width_modulation) and a piezoelectric buzzer. After some research, I found that PWM was able to support 8-bit PCM audio output on the Uno. I was able to find code written for this purpose; I just had to write in the button control and add my own sound data.

The button control proved mostly straightforward once I managed to actually set up the circuits correctly. For the audio, however, I faced a number of constraints. I had to find a balance between the length of the message, the clarity of the actual output, and the strict memory limits of the board. I used espeak to generate the speech, as actual human speech would fare worse in the inevitable lossy compression. I then used a utility called [wav2c](https://github.com/olleolleolle/wav2c) to translate the resulting WAV files into a byte array. There was only space for one message in the board's memory, so that's what it got.


<style type="text/css">
.file-data {
    max-height: 500px;
}
</style>
<script src="https://gist.github.com/colatkinson/9299e8a34fdfc958d103.js"></script>

## The Final Result
While the oracle was damaged in transit to the school, and then again in transit to its final location, it ultimately proved a success. It was placed in a booth, and a cannibalized Staples button was used to activate it. The speakers were a little too soft, especially given the volume of most hallways during Spirit Week, and so if I had more time/resources I would have looked into using PC speakers or an amplifier.

<video src="/assets/videos/VID_20141004_155108.mp4" style="width: 100%" controls />

![The final oracle](/assets/images/IMG_20141009_074612.jpg)
