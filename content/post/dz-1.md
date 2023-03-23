---
title: "IoT Danger Zone - Part 1 - DDoS"
date: 2016-12-16T23:01:08Z
draft: false
cover: "/images/dz-iot/title1.png"
---

# Introduction to DDoS
"The Internet of Things" is a popular phrase these days, and whether people know it or not, they probably have devices in their home that would he considered one of these "Things"

Recently, a huge number of these devices were exploited globally and used to cripple huge swathes of the internet by way of a DDoS (Distributed Denial of Service) attack.

## What is DDoS?

First let me state what a DDoS is not. A DDoS is not a compromise of the users information from the services attacked by a DDoS. It is important to understand this differentiation
from the compromise or 'leaking' type attacks that have made the news lately.

A DDoS is the technological equivalent of gathering a huge crowd of people and filling a supermarket, thus preventing customers from entering. In the meantime,
the products are still safe inside; the crowd aren't taking anything, you're just filling the store with too many people, and assuming they or someone else are not using the crowd as cover to quietly have away with some of the goods while everybody is distracted.

The effects of this hypothetical "filling of the store" can cause two issues; first, the supermarket can not sell anything, this may or may not be the crowds goal.
The other effect of filling the store so packed is that the crowd might knock a few things over, perhaps not intentionally, but they could, by cramming everyone in the store so tightly, cause damage.

## How is DDoS used?

A DDoS does not aid an attacker in stealing user information, not directly anyway, that's the good news, the bad news it that a DDoS can serve several malicious purposes:

1. A DDoS stops the system being attacked from providing the service it provides normally, hence the term "denial of service".
2. A DDoS can be used to distract from other nefarious activities such as attempts to hack and steal information from a system.
3. A DDoS can be used to prevent users or services from protecting themselves. By overwheling a system scanning for malware, said malware could infect services who are unable to find the fix.
4. A DDoS could be used in a manner considered "protest"; by preventing a website from offering its services, it could be considered akin to preventing access to a store by means of a sit-in etc. Note that currently, in the UK at least, DDoS is not legally considered a valid, legal form of protest.
5. Others...?

## But why is it "Distributed"?

If you assumed that there is such thing as a non-distributed denial of service attack, you would be correct.

DoS is the term for a standard denial of service attack, these days however, their damage potential is limited.

The Distributed/Non-distributed part of the name refers to the attackers, not the victims' 'distribution'. 
With a DoS attack, the attacker comes from a single point. Nowadays, blocking a DoS is fairly simple, as it comes from a single point, in this case internet or IP address (See Wiki IP), it can simply be blocked, kind of like someone pressing mute on you when you're shouting too loud for anyone else to talk. 
You keep on talking but they simply won't hear you. Blocking a single address is fairly simple, this is why DDoS became a thing.

## How does it work?

When a company or person runs a service, say this website for example, the system that runs it has a certain number of resources, the machine that the service runs on has a certain amount of memory and processing power, this translates to how many things it can 'process' in a certain amount of time.

Let's say the machine this website runs on can accept 10 viewers every second, because that is the most it can handle with its power, once 10 people have started using this site it can not serve any more people until the existing 10 starts to drop, think of it like a waiter that can only wait on 10 tables at a time.

Once one of the 10 customers leaves, the waiter can serve a different customer, but if there are fewer than (or equal to) 10 customers or users, none of them will see a decline in service.

The concept of service here is similar to that of the waiter, in that, once the waiter has taken your order and delivered your meal (this page), he isn't taking up any time serving you,
and can go serve someone else until you are ready to order something else (load another page). 

Websites are no different, once this page is loaded, the server it runs on is free to go and serve someone else, until, that is, you decide to load another page. 

## The greedy user
Say all 10 slots are filled with viewers or customers, but then a single customer gets greedy and starts taking up the same time as 10 users, the waiter or service that can normally serve 10 people now can only manage 1, and everyone else has to wait.

### How is a user greedy/not greedy?

#### Normal user

Under normal conditions a user isn't generally greedy with their connection to a service, if you load a page such as this one, you are only using the server time while it loads. You can sit here for as long as you like and read the page and that won't put extra load on the server. 
If you decide to switch to another page on the site, you start putting load on it again. If you click a link here to another site, say [Wikipedia](https://en.wikipedia.org/wiki/Load_(computing)) for example, you now start to put load on their server and not this one.

This is all fine, and under normal conditions, browsing the internet like this won't cause any issues.

#### Accidental greed

Sometimes a user's browser or software may malfunction and make lots of connection to a service, this is usually short lived as the user quickly notices and stops it, but still, in internet terms, a single broken user puts very little strain on the average service and would go unnoticed most of the time.

#### Give me attention!

A malicious user will intentionally make lots and lots of connection to a server to constantly get its "attention" stopping it from serving other users, this could be as simple as loading this page too fast, by clicking refresh really really quickly. 
Of course a service designed to handle lots of users can usually handle more than a single human clicking refresh as fast as they can, and the biggest services on the web can handle tens even hundreds of millions of connection per second.

# So how is it done?

Well, since most services can handle a user clicking refresh really quick, an attacker would have to switch to software to do it for them, software can make connection much much faster than a human can hit refresh, but, the machine used by the attacker is just as limited by it computing power as the server it's attacking.

Let's say a single attacked can make 10 connection per second, I could upgrade this site to accept 100 per second, turn the attacker would use only 10% of the sites power, so it could continue serving the other 90% without too much bother.

This is where DDoS comes in, if the attacker has more than one machine, say they have 10 that can each makes 10 connections per second (cps), they can now overcome my upgrade to 100 cps and deny service to my users once again.

If the machines are located in different places (distributed) it makes them 10 times harder to block than a single machine because now there are 10 addresses to block instead of 1.

That's how DDoS gets named.

## Bandwidth

Just like a motorway, there is a certain amount of data (number of cars) that can travel down a connection before things start to slow down, this is because also like a motorway, the "road" must be shared.

A service provider such as this site has a connection with a certain speed, once the speed is all used, users have to queue, like in traffic.

When you purchase an internet connection from a provider, they give you a bandwidth limit, this is the maximum speed you can upload and download things, that is, the speed you can send and receive. It is usually given in a number of Mb/s that is Megabits per second. A megabit being a megabyte times by 8 (there are 8 bits in a byte in this case).

The reason they sell it in megabits is most likely marketing because 32Mb/s looks much nicer than 8MB/s. Note by convention, capital B denotes Byte, lowercase b denotes b. These are often confused.

### How is bandwidth used to DDoS?

Just like a service may have a connection per second limit (cps) it will also have a bytes per second (bps) limit. Say this webpage is 1 MB (1 megabyte) in size, and the connection to it can handle 10 megabytes per second, 10 users can download or view the page per second before they start to see a slower service.

If an attacker can use more than this, just like they can with connections per second, again they can slow access for other users.

Like the cps, large services have more bandwidth available than the average home connection because they have to serve a lot of customers. If this site can handle 100MB/s and an attacker has only 10MB/s bandwidth, again they can not overwhelm the site. By combining 10 machines with 10MB/s bandwidth each however, they can use up all of my 100MB/s and no other users can download or view the page.

## What's this got to do with IoT?

Since an attacker can not easily have the tens, hundreds, thousands, even tens of thousands of devices at their fingertips to overwhelm a large service, they take to hijacking the internet connected devices of innocent victims and using them for their nefarious deeds.

### But how?
Unfortunately not all manufacturers of devices adequately protect the things they sell and unsuspecting users will connect them to the internet to be found by attackers and exploited, sometimes once secure devices have vulnerabilities found in them and exploited at a later date, or perhaps the user has just mis-configured their device due to lack of knowledge or understanding. 

In Pt 2 I'll cover the various things that are exploited, and how they're at fault.
