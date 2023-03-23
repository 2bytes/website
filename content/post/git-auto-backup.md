---
title: "[Linux] Backup Git repos when USB device is inserted"
date: 2015-10-18T00:00:00+01:00
draft: false
cover: "/images/git-bak/title.png"
---

One thing I have always struggled with is finding the time to make backups.

You don’t always want to automate this over the internet for various reasons and might want to backup to an external drive that you can stash away safely.

Recently I decided that my home git server should be backed up and stored elsewhere, off-site.

Making sure to do that regularly can be an effort, particularly so, when you run a headless server. Sure, its as easy as turning on a nearby PC, SSH in and run the backups, but I’m lazy, so there has to be an easier way.

What I’d really like is to plug in a specific USB storage device that I’ve allocated to backups, and when I do so, it will trigger backups automatically and better yet, let me know somehow when it is done, after all, this is a headless server (meaning it has no monitor, or a keyboard and mouse connected).

The answer, in this case is; “udev”. Note, it’s not the only answer, “systemd” is the new kid on the block and perhaps later I’ll update with a systemd implementation.

What I have put together is a way to launch a script when the serial number of a specific USB drive is plugged in and detected. For this, I use “udev” (documentation). “udev” is a dynamic device management tool that allows you to perform actions on device events.

In this case, the “rule”, which is what we call the configuration file that performs the detection waits for a device of the USB kind, checks for a serial number that matches one we set (well find this using some other tools later) and then performs an action we tell it to. In this case, we want it to run a script to backup our git repositories.

The rule can be made to run any script you like, it doesn’t necessarily need to be a git backup, nor need it be a backup script at all. It could he used to start a program like a web browser, or a photo editor when you plug in your camera. Simple replace the git backup script later on with your own, and you’re good to go.


Let’s begin;

*NOTE: Most, if not all of this will require admin permission so make sure you run as “root” by performing “sudo su” to get a root prompt or putting “sudo” in front of each command.*

# Find the serial number of your device

Assuming you’re on a standard,non-headless Linux machine( i.e. Raspberry Pi running Raspbian with a monitor connected), your USB drive should be mounted automatically when you plug it in.

What you need to know, is where it has been mounted and name the kernel has given its device.

we can check this using `df`;

``` bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb3       111G   15G   92G  14% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev            7.8G  4.0K  7.8G   1% /dev
... [snipped]
/dev/sdc1        15G  328K   15G   1% /media/hamid/1686-EE80
```

As we can see, amongst others, my 16GB pen drive has been mounted at /media/hamid/1686-EE80 and its device name is /dev/sdc1. Now, armed with this information, we can get its USB device tree by passing the name “sdc” to “udevadm”.

``` bash
$ udevadm info -a -n sdc
```

Udevadm info starts with the device specified by the devpath and then
walks up the chain of parent devices. It prints for every device
found, all possible attributes in the udev rules key format.
A rule to match, can be composed by the attributes of the device
and the attributes from one single parent device.

``` bash
looking at device '/devices/pci0000:00/0000:00:14.0/usb1/1-5/1-5:1.0/host9/target9:0:0/9:0:0:0/block/sdc':
    KERNEL=="sdc"
    SUBSYSTEM=="block"
    DRIVER==""
    ATTR{ro}=="0"
    ATTR{size}=="31266816"
    ATTR{stat}=="459 15168 16871 1020 1 0 1 0 0 540 1020"
    ATTR{range}=="16"
    ATTR{discard_alignment}=="0"
    ATTR{events}==""
    ATTR{ext_range}=="256"
    ATTR{events_poll_msecs}=="-1"
    ATTR{alignment_offset}=="0"
    ATTR{inflight}=="       0        0"
    ATTR{removable}=="0"
    ATTR{capability}=="50"
    ATTR{events_async}==""

  ... [snipped]

looking at parent device '/devices/pci0000:00/0000:00:14.0/usb1/1-5':
    KERNELS=="1-5"
    SUBSYSTEMS=="usb"
    DRIVERS=="usb"
    ATTRS{bDeviceSubClass}=="00"
    ATTRS{bDeviceProtocol}=="00"
    ATTRS{devpath}=="5"
    ATTRS{idVendor}=="0781"
    ATTRS{speed}=="480"
    ATTRS{bNumInterfaces}==" 1"
    ATTRS{bConfigurationValue}=="1"
    ATTRS{bMaxPacketSize0}=="64"
    ATTRS{busnum}=="1"
    ATTRS{devnum}=="8"
    ATTRS{configuration}==""
    ATTRS{bMaxPower}=="200mA"
    ATTRS{authorized}=="1"
    ATTRS{bmAttributes}=="80"
    ATTRS{bNumConfigurations}=="1"
    ATTRS{maxchild}=="0"
    ATTRS{bcdDevice}=="0201"
    ATTRS{avoid_reset_quirk}=="0"
    ATTRS{quirks}=="0x0"
    ATTRS{serial}=="211322437220FA19914D"
    ATTRS{version}==" 2.00"
    ATTRS{urbnum}=="1509"
    ATTRS{ltm_capable}=="no"
    ATTRS{manufacturer}=="SanDisk"
    ATTRS{removable}=="removable"
    ATTRS{idProduct}=="5530"
    ATTRS{bDeviceClass}=="00"
    ATTRS{product}=="Cruzer"
```

As we can see from the output of udevadm the USB pen drive is at /dev/sdc.

Most of the information shown is not useful to us, but a couple of key pieces we can use to make sure we detect the correct drive. We may, for example, have 3 or 4 of the same brand and model USB drive, but only want it to detect the correct one.

We can see from the bottom tree, the device (manufacturer) is a “SanDisk” and its model (product) is “Cruzer”. The really important bit is the serial (serial) “211322437220FA19914D” (note, I have mangled the real serial a bit to hide it, but the structure has not changed).

If you really feel like it, you could use other attributes like size, or model (for example if you wanted your rule to run for all “Cruzer” devices you plug in) to device which devices cause our command to run from the udev rule.

# UDEV

You need to create a file in the udev rules directory; In my case I called it “90-run-on-usb.rules”. The name is mostly unimportant, BUT, the number at the start determines how early or late it runs in relation to other rules in there.

``` bash
nano /etc/udev/rules.d/90-run-on-usb.rules
```

Inside your rule file add the following (it has to be on a single line);

``` bash
KERNEL=="sd[a-z][0-9]", SUBSYSTEMS=="usb", ACTION=="add", ATTRS{serial}=="211322437220FA19914D", RUN+="/usr/local/bin/usb-chainload.sh '%k'"
```

The `KERNEL` section says to look for a device beginning with `sd` follows by another letter from a-z (not capitals) and finally ending in a number between 0 and 9. This snytax for finding things is called a Regular Expression. If your device is given a different naming scheme by your kernel, e.g. hda. adjust the rule accordingly.

`SUBSYSTEMS` is telling udev to look through the entire tree of subsystems (note it is plural) for one on the `usb` bus.

`ACTION` ensures we run the rule only for a specific action; Without this, the rule would run both when the device is added and when it is removed. You may want to run something on remove in some cases, and as such you can add a second line to run a remove action. In this case we are interested only in the add.

Now for the cool stuff; `ATTRS{serial}` checks the device’s serial attribute and matches against our serial (note the double equals).

Finally, we run a script `usb-chainload.sh ‘%k'`.

Ok, so you noticed the `%k` and are wondering what that is for? Well, because the device name assigned to a drive when it is plugged in is dynamic, we need to make sure to tell our chainload script the correct drive to work with. In this case, we intend to use the drive to update some git repository backups stored on it. If you don’t need to work on the drive itself, feel free to leave this part out.

# UDEV update rules

When you create or modify a udev rule, its important to tell udev to refresh. This is done by running the following command.

``` bash
udevadm control --reload-rules
```

# IMPORTANT

*One of the critical mistakes I have noticed when searching the internet for how people do this is that they misunderstand how udev rules work. If you call a <s>rule</s> script (Correction, 2016) that will perform some long running tasks from your rule, the rule will not “complete” or return, until the script has finished.*

*The solution is to chain-load your script. Essentially this means to start as script that will start your main script in the background and “return” immediately to tell udev it has run successfully.*

# Chainloader
``` bash
#!/bin/bash
/usr/local/bin/backup-git-repos.sh $1 &
exit 0
```

# Git Backup

``` bash
#!/bin/bash
log=/tmp/usb-runner.log
notify=false
TIMEOUT=10 # Timeout in seconds while waiting for the device

##### DO NOT MODIFY AFTER THIS POINT #####

now=$(date +"%T")
end=$((SECONDS+$TIMEOUT))

echo "START.....: $now" > $log

dev=$1

echo "DEV: $dev" >> $log

if [ ! $dev ] ; then
        echo "Device not found [$dev]" >> $log
        exit 1
fi

until (df /dev/$dev 2> /dev/null | grep $dev ) || [ $SECONDS -ge $end ] 
do
        echo "." >> $log
        sleep 1
done

df /dev/$dev 2> /dev/null | grep $dev &> /dev/null

if [ $? -ne 0 ] && [ $SECONDS -ge $end ] ; then
        echo "Timed out; Device not found"
        exit 1
fi

echo "Device ready" >> $log

dir=$(df /dev/$dev | grep $dev | tail -1 | awk '{ print $6 }')

echo "Device...: $dev" >> $log
echo "Mounted..: $dir" >> $log

cd $dir

for d in `ls`
do
        (cd $d && git remote update) >> $log
done

if [ "$notify" = true ] ; then 
        echo    "SENDING NOTIFICATION" >> $log
fi

later=$(date +"%T")
echo -e "DONE......: $later\n" >> $log
```

This script takes the device name as a parameter. You can (and should) change the log location as /tmp will be erased each reboot. Timeout can be changed as required. Notify can also be changed. In this script it does nothing, but you can use the notify check near the bottom to send yourself a push message when the script completes using for example one of my other scripts like [Bashover](/post/bashover) or [Bashbullet (Git)](https://github.com/2bytes/Bashbullet).

The script `backup-git-repos.sh` can be replaced with any script you want to run, and the `$1` is only needed, remember, if you want to pass a parameter to the new script.

# Prepare pen drive

Now, in order for your repositories to be updated, you must first perform an initial clone on to the pen drive. Change directory in your terminal to the root of your pen drive and run the following command for each repository you want to backup.

``` bash
git clone --mirror <repository address>
```

After your are done, you should have a bunch of .git directories in the top level of your pen drive.

Now, each time you plug in the device, the script will run through those directories and update them all. The beauty of git is that the backup does not need to be run from the same server that hosts your git repositories, but any machine that can clone them.