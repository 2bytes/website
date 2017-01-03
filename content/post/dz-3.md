+++
draft = false
date = "2017-01-03T21:41:40Z"
title = "IoT Danger Zone - Part 3 - Firmware"
image = "dz-iot/title1.png"
+++

> Note: I'm going to put a few links in this post to definitions, Wikis and/or articles, feel free to detour there and read more depth about anything of interest that I don't cover in details here.

Most modern devices that connect to the internet require some form of software to run. 
In home electronic devices such as routers, security cameras and even smart phones this software is also known as [firmware](https://en.wikipedia.org/wiki/Firmware). 

Firmware usually can't be modified by the user during the normal operation of the device. 
It is written and installed by the manufacturer (or someone else perhaps) to provide the features the device is designed for. 
Some manufacturers provide updates, others do not. When they do, it is usually via some interface or application that allows the update to be sent to the device.
With some modern devices such as 'Hue' devices, this can be done with a mobile phone application. 
With others, say an Xbox controller you may need to connect it to an Xbox or PC using a USB cable. Did you know Xbox controllers have [upgradable software](http://support.xbox.com/en-GB/xbox-one/accessories/update-controller-for-stereo-headset-adapter) in them?
 
Your home router for example may have a web page that you can select a file from to update it with, and you would have obtained this file by searching
the manufacturer website for updates for your exact model.

The older, cheaper, or less popular the device, the less likely it is to receive updates. If it's "branded" by some no-name company, or does not have an obvious
company brand, it likely also won't get updates, or if it does, you likely won'y be able to find them.

# Bugs

Software can inevitably have bugs and sometimes these bugs can cause vulnerabilities in the devices that attackers can exploit.

More popular devices mean more vulnerable devices to be attacked. Once a vulnerability has been found for a particular device, all units of that devices everywhere are vulnerable, assuming they all run the same un-patched software.

The only way to resolve the issue is to replace the software or replace the device. Some devices, if you're lucky, may have the ability to disable a particular feature that is vulnerable
and you may get a little more mileage out of it before having to dispose of it (responsibly).

# Updates

Some manufacturers update their firmware to add new features or to fix bugs, sometimes in response to security issues, other times for more trivial things.

Very popular devices, say for example 'Hue' light bulbs might receive more attention from a manufacturer simply because of economics.
If something sells well, they can put more time and money into developing it further.
Sometimes it just pays to pay more for things, and the 'Hue' products will get a lot more attention than some random no-name device off of ebay.

Bear in mind, even big, experienced companies make mistakes, and sometimes broken updates can be released, keep an eye out for fixes soon after updates as users start to notice problems.

If you see any obvious problems, go to the company support page and report the issue, if it's a bug, it's probably fixable.

# Automatic vs Manual

Most devices that have firmwares do not update automatically,  often it is up to the user to visit the manufacturer website and find the update and follow instructions to install it.

Rarely devices will update themselves automatically but even those that do will require consent from the user before this will happen.

Some home routers from large providers will automatically update based on remote commands from their provider, these devices are usually not ones you "own" but belong to the service provider, and what they do or don't do is usually out of your control.

The real question here is should I update manually/automatically and/or at all? Let me take the long-winded approach to answering this because the answer is not black and white.
 
Automatic updates are great, you turn them on, you forget about them, and your things will get updated when the manufacturer releases updates. Manual updates take effort, you have to remember to do them, and further you have to be aware or on the lookout for issues cropping up.

The problem with automatic updates is that you take out the human component. 

Let's say a vulnerability allows an attacker to hijack the update connection and replace whatever was going to be downloaded from the manufacturer,
with a malicious software/firmware to suit their nefarious needs. Your device will automatically install this and you'll never (most likely) know it happened. Now your device is under their control.

So manual it is? Well, yes,ideally, but this means taking time out of your schedule, regularly or keeping on top of tech news to know when one of your devices is affected.

What this does mean, is that you are in control of when things get updated, refer to the "Release notes" that the manufacturer should publish to tell you what has changed, and make sure you're happy with the effects before updating. 
If you're not happy, turn the device off, and/or disconnect it from the internet until you can ask somebody either from the manufacturer or tech community who can answer your questions. 

Performing updates manually means you can also test your device and be happy it is still working correctly before moving on and forgetting about it.

# Obsolescence

An often overlooked fact with smart devices is that even manufacturers who provide regular updates for their products have to stop providing them at some point.

Whether by economics or simply because they want you to buy their new thing, the case is probable that that expensive WiFi router you bought back in 2013 still appears to work great, however, due to things like [Heartbleed](https://www.heartbleed.com) and [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)) should now be considered broken or obsolete and discarded because at 3 years old to date the manufacturer doesn't have the time or resources to keep it updated, particularly when they might release 5 or 6 or more devices a year.

The internet and the things connected to it and via it are a rapidly changing landscape, and there is no choice, if security is to be maintained, but to accept this fact
and if or when necessary simply dispose of devices that are no longer supported or capable of maintaining a sufficient level of security.

# Unfixable

There are devices that are so broken in their software or hardware design that they simply cannot be fixed, 
either due to a flaw in the hardware design meaning an update can't solve a vulnerability, 
or because of a part of the software where the bug is found is read-only and can't be updated.
 
Although it is rare for modern devices not to have the ability to update all of its software, heed the warnings, 
if the general recommendation on the internet is to 'turn it off'. Turn it off and dispose of it, 
perhaps look into warranty for newer devices, but don't just ignore it.

# Botnets

The software aspect of these devices are what make them vulnerable, the fact that they can be made, one way or other, to execute ('run') code they were not built for, 
makes them prime devices for creating botnets and performing DDoS's with them. 

Sometimes even worse attacks can be performed for devices with crossover into the physical world such as life-saving devices (think; medical machines) or door locks, 
these devices, with good reason, should be treated with nothing short of complete paranoia.

Read about [Stuxnet](https://en.wikipedia.org/wiki/Stuxnet) to understand what I mean by the crossover between these software-running devices and the real world and the potential effects
a 'worm' can have on the devices it infects and the world around it.

# Open source

Open source is, in my opinion, one of the greatest things to happen to software, ever. What open source means is that the people who write code, make the code available to the general public under some form of licence.

There are many open source licences, and I won't cover them here, but what it does mean is that you, yourself, can look at the code that your open source device runs. For example; the Raspbian operating system/software that the Raspberry Pi runs is open source.

Raspbian itself is based on the source code for another open source operating system, [Debian](https://www.debian.org/). You can get obtain the code, build it yourself (perhaps with some difficulty) and run it.
Importantly, this means the code can be audited, and with millions, instead of tens or hundreds of eyes on the code, it makes it much easier to find problems, and fix them; quickly.

Don't forget, this also means that if you can read the code, and build it, so can an attacker, but the advantages of open source far outweigh the security risks. 
Security is not truly security if it's being obtained by "obscurity".
The strongest, most secure code is that which is openly readable and visible by all, but still stands up to attack.

