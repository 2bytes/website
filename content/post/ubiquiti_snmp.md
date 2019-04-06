---
title: "Ubiquiti SNMP v3 + Grafana + Collectd"
date: 2017-08-06T23:20:53+01:00
draft: true
image: "ubnt/title.png"
---

I'm going to try keep the commentary short with this post, as I think the information is more important to the type of people interested.

The goal of this post is to (initially) give you this;

![Grafana dashboard of the EdgeRouter Lite 3](/images/ubnt/erl.png)

and this;

[EDGESWITCH GRAFANA PICTURE]

There are several requirements to get to this, and I would like to first of all point out, 
that how I approach this is by no means the only way.

I run an EdgeRouter (EdgeMax, Vy(atta) based OS) and an EdgeSwitch (Fastpath?), even though the devices are in the same line, they run different OSs.

What this means for you and me, is that configuring them is completely different; Hooray.

I also run Unifi AP-AC Pro Wifi APs, as far as I can tell at the moment, these require the UniFI controller to be present
in order to access any useful stats, so I haven't ventured there yet.


# Components

These are the components of my setup:

1. Hardware device I want to see stats from (EdgeRouter, EdgeSwitch)
2. Database to store the SNMP data so I can see historic (I don't want to be watching the graphs 24/7)
3. Software running somewhere to collect SNMP
4. Software to display the graphs

## 1 - Hardware
* Hardware is easy; maybe. I can't make any assurances here that particular devices do or do not have SNMP v3 support.
* I'm not here to explain how SNMP v3 differs from v1 or v2, but it does add more security.
* I __can__ confirm that my _EdgeRouter-Lite 3_ and  EdgeSwitch-8-150W do support SNMPv3.

## 2 - Database
[InfluxDB](https://www.influxdata.com/time-series-platform/influxdb/) is what I use; That's because I already use it for sensor data my MQTT broker is collecting (that's for another post). 
The key here is that InfluxDB, unlike other database alternatives such as relational databases (MySQL, PostgresSQL, Sqlite et al) is
that `InfluxDB` is a [time-series database](https://en.wikipedia.org/wiki/Time_series_database). I also run this in a Docker container 
(I'll post the Dockerfile for this).

## 3 - SNMP Collector
I use an old-school favourite called [collectd](https://collectd.org/). I run it in a Docker container (I'll also post the Dockerfile).
You could use something like [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), or I'm sure other collectors, in this case it'll need to support InfluxDB.

## 4 - Graphs
Last but not least is the graphing, here I use [Grafana](https://grafana.com/); why? because it's beautiful, why else?
Grafana is designed for graphing time-series data and is very easy to configure to retrieve its data from InfluxDB.

With InfluxDB as a backend, it supports powerful GUI based query configuration so that you can not only show things like
the total data that has flowed through a port, using the cumulative value the hardware supplies, but also data throughput, instantaneous data rates,
averages, mins, maxes, etc, etc (Yep, Docker, yep, will post).

# Steps

## 1 - Enable and configure SNMP (v3)

For my purposes I configured SNMPv3 with authentication and encryption enabled.

Unfortunately, on the Ubiquiti front, there isn't a lot of great documentation on how exactly to do this, one thing however is certain;

> The WEB UI does not support SNMP v3 configuration. Do it in the shell.

### 1.1 - EdgeRouter

First of all we'll need to connect to the shell on our EdgeRouter, of course, if you don't have an EdgeRouter and just want 
to configure an EdgeSwitch, just skip to 1.2 below. Shell is obtained either by SSH, telnet, or using the "CLI" option
in the UI;

Once in the shell, enter configuration mode and edit the snmpv3 service.

```bash
user@ubnt:~$
user@ubnt:~$ sudo bash
root@ubnt:# (note we're now root)
root@ubnt:~$ configure
root@ubnt# edit service snmp
[edit service snmp]
root@ubnt#
```

Next, we need to prepare some information in order to configure the router. First, we will need two pass keys.

Generate and store 2 different pass keys to be used for authentication and encryption, you'll need them again later.

Also, we will need an Engine ID. [This (PDF)](http://docs.logmatrix.com/nervecenter/guides/NC-SNMPv3-EngineIDs.pdf) is the best source I've read to describe EngineIDs,
the different types of them, and how to generate them. This one is important, because it is used to encrypt the pass keys you will supply in plain text.

If this is not set, encrypting of your keys will fail.

Now we need to configure snmp as follows; fill in the blank areas, and replace VIEW and USER with a name of your choice, if desired.

Listen address should be set to the IP on your LAN that the ERL will listen for SNMP queries on.

> Before you continue, go to the web UI of your EdgeRouter, enable SNMP Agent check box and leave the rest blank, then save.

```bash
user@ubnt# set listen-address <listen address here>
[edit service snmp]
(the remaining settings are v3 specific)
root@ubnt# set v3 engineid 0x<hex engine id>
root@ubnt# set v3 view VIEW oid 1
root@ubnt# set v3 group SNMP view VIEW
root@ubnt# set v3 group SNMP mode ro
root@ubnt# set v3 group SNMP seclevel priv
root@ubnt# set v3 user USER group SNMP
root@ubnt# set v3 user USER mode ro
root@ubnt# set v3 user USER auth type sha
root@ubnt# set v3 user USER auth plaintext-key <auth password>
root@ubnt# set v3 user USER privacy type aes
root@ubnt# set v3 user USER private plaintext-key <enc password>
root@ubnt# commit
```

The next part is important so we're going to double check everything;

First we need to exit after the commit or the command will not work, make sure you ran the commit above first!

```bash
root@ubnt# exit
[edit]
root@ubnt# /opt/vyatta/sbin/vyatta-snmp-v3.pl --update-snmp
[edit]
root@ubnt# show service snmp
```
You should now see something like this; Note there are `-` (minus) signs next to the plaintext-key and `>` (grater-than)
signs next to the encrypted-key. This indicated the ones with minus will be removed, and the ones with greater-than
will be added on next commit.

```bash
root@ubnt# show service snmp
 listen-address <listen address> {
 }
 v3 {
     engineid 0x<engine id>
     group SNMP {
         mode ro
         seclevel priv
         view VIEW
     }
     user USER {
         auth {
>            encrypted-key 0x<encrypted auth key>
-            plaintext-key <auth password>
             type sha
         }
>        engineid 0x<engine id>
         group SNMP
         mode ro
         privacy {
>            encrypted-key 0x<encrypted enc key>
-            plaintext-key <enc password>
             type aes
         }
[edit]
```

If it all looks good, run commit, then save, and you're good to go:

```bash
root@ubnt:# commit
[edit]
root@ubnt:# show service snmp
... (double check the plaintext-keys are gone from the output).
root@ubnt:# save
Saving configuration to '/config/config.boot'...
Done
[edit]
root@ubnt:# exit
root@ubnt:/home/user/# exit
user@ubnt:~$
```

If something went wrong and you want to start again, wipe out the SNMP config:

```bash
user@ubnt:~$ sudo bash
root@ubnt:/home/user/# configure
[edit]
root@ubnt:# delete service snmp
[edit]
root@ubnt:# show service snmp
(everything will have a minus next to it)
root@ubnt:# commit
[edit]
root@ubnt:# save
Saving configuration to '/config/config.boot'...
Done
[edit]
```

Now you can start again.


Once completed, you can use the following command on a linux machine to test;

```bash
snmpwalk -u USER -A <auth pass> -a SHA -x AES -X <enc pass> -l authPriv <listen address from previous step>
```

### 1.2 - EdgeSwitch

[TBD]

### 3 - Database

[TBD]

### 3 - SNMP Collector

[TBD]

### 4 - Graphs

[TBD]