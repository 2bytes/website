+++
date = "2013-11-26T00:00:00+01:00"
draft = false
title = "Pixel Commander - Arduino UART RGB L.E.D. Controller"
image = "pxl-com/title.jpg"
+++

![9-LED Pixel Square with Arduino Compatible](/images/pxl-com/title.jpg)

**UPDATE 2016: This project is not dead, but, technology has improved since I started the project and there are now several alternatives;  
First of all, the APA102 RGB L.E.D is much easier to work with (SPI, so it'll work on the Raspberry Pi's non-realtime OS) and also, [FastLED](https://github.com/FastLED/FastLED) is a much better way of managing L.E.Ds**

Pixel Commander is part of a much larger project I’m currently working on. Its purpose is to allow me to remotely control one of the LED boards shown in the image (the board design will also be open sourced when it is complete).

Pixel Commander takes commands over UART and uses them to change the state (colour, brightness) of the LEDs on the board.

Most of the heavy lifting in controlling the LEDs is done by the Adafruit NeoPixel Library which you will need to obtain and install into your Arduino development environment before you can compile this sketch.

These aren’t ordinary RGB LEDs, which is why the Adafruit Library is required. They are, individually addressable, data controlled one wire “pixels” capable of displaying over 16 million colours each (theoretically).

As of writing, I have 4 simple commands written for the sketch, each has to be sent to the arduino over serial in hex form;

Set Colour
``` c
50 43 53 43 RR GG BB 00 00
```
Where RR is the Red byte, GG is the Green byte and BB is the Blue byte. RRGGBB together form a standard six digit hex colour code.

Set brightness
``` c
50 43 53 42 XX 00 00
```
Where XX is the brightness from 00 (Off) to FF (Max brightness).

Brightness Level
``` c
50 43 42 4C YY 00 00
```
Where YY is one of the following:

* Brightness Up: 55
* Brightness Down: 44
* Brightness Max: 49
* Brightness Min: 4F

Brightness up/down will raise or lower the brightness by 15 (out of 255) each time.

Identify
``` c
50 43 49 44 00 00
```
Identify will cause the LEDs to switch to white and pulse twice before returning to their previous state.

This last command is used to identify a “light” visually once they are all connected wirelessly to the IoT project controlled by a Raspberry Pi base station which takes commands from a mobile app by REST API. With several of these lights dotted around a room or building, this gives the easiest way to visually identify them all during setup.

As the project progresses I intend to make lots of enhancements, see the Bitbucket for status and to do list.

The source code is available on [Github](https://github.com/2bytes/PixelCommander)


