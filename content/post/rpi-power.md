---
title: "Raspberry Pi B vs B+ Power"
date: 2014-07-21T00:00:00+01:00
draft: false
cover: "/images/rpi-pow/title.jpg"
---

![Rapberry Pi Model B and Raspberry Pi Model B+](/images/rpi-pow/rpi01.jpg)

With the recent release of the Raspberry Pi B+, I was lucky enough to be working right next to Westminster bridge where the very kind Element 14 were giving out free Raspberry Pi B+ and Ice Cream, both of which I managed to get my hands on, thanks guys!

One of the key factors that interested me with the new Pi was its move to more efficient switching regulators instead of the old linear (read: inefficient) ones it had before. This move, the Raspberry Pi Foundation tell us lowers the power consumption of the little board noticeably.

Being that I tend to run my Pis 24/7 (I now have a Pi Running OpenVPN, and a Pi running Seafile), the smallest difference with this relatively low power device should be significant.

Seafile had proved difficult on the B, as there are difficulties powering external HDDs from its restricted USB current output, the B+ promised to solve this, and I can tell you that it delivers! Read on for the numbers!

# Setup - Common

* Standard Raspbian Wheezy image (2014-06-20)
* Logitech Wireless Keyboard/Mouse dongle (old, large, long wire, uses lots of power)
* Edimax WiFi N Micro-Dongle
* 1.8M HDMI cable to LCD Monitor

# Setup - Different

* Raspberry Pi Model B and Raspberry Pi Model B+
* SD in Model B vs microSD in B+ (this could potentially be negated by using an adapter to switch the microSD into the Model B)

# Test 1 - Graphical Desktop
The first test shows both Pi’s idling on the graphical desktop (running startx from the terminal after logging in). At this point, both Pi’s are using WiFi connected to a WiFi N network in the next room, with the USB KB/Mouse dongle attached and HDMI to a monitor.

![Rapberry Pi Model B and Raspberry Pi Model B+ Graphical Desktop](/images/rpi-pow/rpi02.jpg) Model B, left, and Model B+ right.

The little 2bytes logo in each image reminds me these were running the graphical desktop as I didn’t manage to get the display into the image!

## Voltage  
- B   : 5.34V  
- B+: 5.33V

## Current  
- B   : 408mA  
- B+: 304mA

## Power (= Voltage x Current)  
- B   : 2.18W  
- B+: 1.62W

NOTE: Ignore the mAH reading on the meter, this is the total charge that has flowed since the last time I reset the device. This kind of reading is useful to see how much battery power you consume over time!

# Test 2 - Stress
This test involves running the Pi, without a graphical desktop, but instead using the “stress” tool (See here) to max out the CPU of the Pi to 100% and artificially fill up RAM.
 
This is not of course an exhaustive way to test out the Pi under full load, but it does give a reasonable indication of the power consumption when the CPU is under heavy load and RAM being filled. This also doesn’t account for GPU use, but perhaps an update can cover that with some rendering tests etc.

![Rapberry Pi Model B and Raspberry Pi Model B+ Graphical Desktop](/images/rpi-pow/rpi03.jpg) Model B, left, and Model B+ right.

## Stress
- 1 CPU worker thread
- 128MB memory allocation (-m flag)

## Voltage
- B   : 5.35V
- B+: 5.31V

## Current
- B   : 502mA
- B+: 322mA

## Power (= Voltage x Current)
- B   : 2.68W
- B+: 1.71W

As can be seen, even under load, the B+ keeps its cool weighing in below the 2W mark for a whopping 0.91W saving over the B!

# Test 3 - External HDD
This final test does not include the model B because frankly it’s just not possible. Here, the B+ is directly powering a WD Passport 500GB external USB HDD.

![Raspberry Pi Model B+ with External USB HDD ](/images/rpi-pow/rpi04.jpg)Model B+ & HDD

## Software
Seafile server hosting a Seafile cloud (because i’ve had much better results than with Owncloud).

## Voltage
- B+: 5.34V

## Current
- B+: 510mA

## Power (= Voltage x Current)
- B+: 2.73W

## Notes
- This required the special extra USB current config flag (intentionally not printed here because it should be used with caution).
- A 1A PSU refused to spin up the HDD even with the extra current flag (yes, even though you can see it’s clearly using less than 1A total, that’s because the initial start of the drive requires a peak voltage/current to succeed).
- The drive is spinning and not at rest or in standby when taking the reading.

# Conclusion
As can be seen from these quick tests, there is certainly an improvement in power consumption due to the new switching power supply in the B+, and coupled with the USB current increase, the Pi can now be used with external HDDs much more easily without the need for powered hubs or spliced cables.