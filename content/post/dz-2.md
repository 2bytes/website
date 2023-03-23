---
title: "IoT Danger Zone - Part 2 - IP Cameras, baby monitors, and smart kettles"
date: 2016-12-18T13:01:37Z
draft: false
cover: "/images/dz-iot/title1.png"
---

There are a growing number of devices in the home that connect to the internet, some of the most prolific in terms of remote attack are those designed to give the user access remotely, perhaps from their mobile when they are away from home. 

Devices such as DVRs (Digital Video Recorders) and security cameras that allow you to check on your home while you're away, do so by opening a connection out to the internet that you can connect to remotely. 
Unfortunately this means that if you can connect, so can an attacker.

# But attackers have to find it, don't they?

## Yes, but no

Well usually yes. Normally, an attacker would need to scan through all of the billions of possible IP addresses to find one running a security camera on the default port in order to exploit it.

> IP (Internet Protocol - See [Wiki](https://en.wikipedia.org/wiki/IP)) is a numeric identifier (a little like a phone number) that is assigned to every device that connects to the internet
> e.g. You home router.


But, as the user of a company's product, would you like to have to remember that to connect to your camera from you phone you must connect to '178.62.35.80'? Or would '2byt.es'
be easier to remember?

Because a lot of manufacturers want your life as a customer to be easier, and rightly so, they will ship your device with something called Dynamic DNS, this means they will provide all of the devices they sell with an easy to type, easy to remember name that allows access to it on the internet.

> DNS (Domain Name System - See [Wiki](https://en.wikipedia.org/wiki/Domain_Name_System)) is a system that converts human readable domain names '2byt.es' to machine readable IP addresses '178.62.35.80'.

The way this is done is by giving each device a unique identifier say 'cam0001' and then having the device tell the manufacturer website where it is periodically.

The manufacturer of this camera will put a special website up, with unique names for each camera, say "cam0001.cameracompany.com", this is nice and easy(ish) for you to remember
and your camera, when connected to your home network will periodically tell cameracompany.com that it can be found at IP address 'A.B.C.D'

Say your address is "1.2.3.4" the camera will tell it's manufacturer that it can be found at "1.2.3.4".

When you want to access you camera remotely, the app on your mobile for example, using the login details you have will ask the manufacturer, hey, I want to see the video from cam0001 it can be found at the address "1.2.3.4".

Because of the DNS they have set up for you, you will only need to type the easier, human readable "cam0001.cameracompany.com" as the address, then enter the login and password.

The manufacturer site will oblige since you proved who you are by logging in and show you the video?

## So what?

Well, let's say an attacker knows that a manufacturer X makes a camera and uniquely identifies them using a series of numbers say from cam0001 to cam9999, an attacker can test all 9999 cameras to see if they exist and are online by asking the manufacturer site to show it the video.
The problem here is that unlike scanning all of the IP addresses on the internet, they know that behind these addresses exist a specific manufacturer's cameras. 
If there is a known flaw in the cameras from that manufacturer, they can potentially access them. 


# But they don't know my password, do they?

Well, again for various reasons this is a murky answer. 
Let's say manufacturer X has a camera that is known to have a default username and password of "user" and "mycamera", the average user might not change the password and so the attacker can just test every camera name until it finds the ones that haven't changed the default password and voila they can now see the video from that camera, easier than you thought eh? 
Sadly it really is this easy. Most manufacturers, wanting to make your life easier will set a default password, some, being better than others will at least make it unique to your device, and print it on the device along with the serial number.

## Raspberry Pi

One prime example of a device that follows this pattern of insecurity is the Raspberry Pi. The Raspberry Pi is designed for learning, and the people behind it want it to be easy to use.
All Raspberry Pi's running the default (official) software have a username and password combination of "pi" and "raspberry".

But, before you go tearing that new Raspberry Pi out of the hands of your children, let me explain.

The Raspberry Pi, although designed to connect to the internet, does not by default, expect incoming connections from the internet.

Also, the Raspberry Pi team recently updated their default OS recently to disable SSH ([See post on Raspberry Pi blog](https://www.raspberrypi.org/blog/page/2/?fish#a-security-update-for-raspbian-pixel)) by default.

Now imagine you configure your Pi to be a camera, using the RaspiCAM add-on, like the one from company X, and connect it to the internet to enable remote access. Somebody that knows your IP and that you have a Pi camera can connect and view your video.

They know your username is 'pi', and that your password is 'raspberry' unless you changed it, which you did, didn't you?

So, if you and/or your children use Raspberry Pis, you are likely safe, but, I implore you to encourage security from the get-go by making the process of changing the default username and password part of the learning process.

Ingrain this in your setup process of new devices as a given; Always change the username and password, and always use a different password.

This is an easy process to sell to your children as a "customisation" step, letting them create their own unique username and password helps make the device theirs,
and gives them some responsibility over remembering and/or safely storing their credentials.

# How can attackers get in if they don't know my password?
It has been known that certain poorly coded devices will allow access to certain features, sometimes even critical ones without asking for the password if the attacker knows where to look, perhaps by changing a certain value in the address of a web connected device or simply by browsing to a non-advertised page on the device.
Some low quality home routers offered by certain internet service providers are particularly vulnerable to this, the EE Brightbox for example (See [BBC coverage](http://www.bbc.co.uk/news/technology-25809208)) which I myself was given when joining them. Worse yet, even after a security patch it was still vulnerable.
More recently, Netgear routers found to have critical vulnerabilities, the only solution? "Pull the Plug" (See [Wired Coverage](https://www.wired.com/2016/12/ton-popular-netgear-routers-exposed-no-easy-fix/))

For reference, the EE Brightbox router I was given was never used it my home, I connected it to confirm vulnerability and disposed of it. 
This may sound knee-jerk or extreme but I assure you, it is not. 
When signing up to an ISP, search the web for "vulnerability" or "exploit" and the model number of the router, if affected, replace it or contact your ISP immediately.

# What else can an attacker do?

Well, if an attacker can log in, they can change settings, imagine instead of a security camera, this is your heating system, now an attacker can crank up the heat to maximum, best case, they cause you a huge energy bill, worst case someone could be seriously harmed, or your home burned down.

As the number of connected devices grows, so does the risk; With connected ovens, fridges, washing machines, heating, lighting and much more, it is prudent to be secure.

# But is this really a problem?

Yes! There are several well known manufacturers that use the same default usernames and passwords on their devices, worse, some, in 2016 have been known to accept blank usernames and passwords even when a user has set one, because of poor coding.

There was a recent smart kettle (See [The Register](http://www.theregister.co.uk/2015/10/19/bods_brew_ikettle_20_hack_plot_vulnerable_london_pots/)) that is known to leak WiFi password to passers-by scanning for them, giving them unfettered access to your home network once the password has been overheard.

The recent, late 2016 DDoS (See [The Guardian](https://www.theguardian.com/technology/2016/oct/26/ddos-attack-dyn-mirai-botnet)) that took out huge portions of the internet with a 1.2 Tb (1.2 Terabit; 1 megabit x 1024 = 1 Gigabit, 1 Gigabit x 1024 = 1 Terabit), that's over a thousand, thousand megabits of data in a second! Compare to the numbers in [Pt 1](https://2byt.es/post/dz-1/) for why this is a problem!

Pt. 3 Will cover Firmware/Software and why it's difficult to keep these devices secure and up to date.