---
title: "vcgencmd for Ubuntu 20.04 aarch64 on RPi 4B"
date: 2021-01-03T17:54:42Z
draft: false
---

### Happy New Year!

This post is a quick one (hopefully) to help those like myself, running Ubuntu on their Raspberry Pi, get access to the `vcgencmd` tool in order to check things like throttling state, clock frequencies and temperature.

<!--more-->

> **UPDATE 2021-04-20:**
> Thanks to user [@lucasfolino on Github](https://github.com/2bytes/website/issues/28) for pointing out that `vcgencmd` (amongst other utils) can be installed by adding the package
`libraspberrypi-bin`. If you want the quickest and easiest way to get vcgencmd, simply run
`sudo apt-get install libraspberrypi-bin`, that's it, done!

## Backstory

I spent the end of last year working on a new cluster set-up for my Raspberry Pi 4s, the start of the previous - was documented on the [Raspberry Pi forum](https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=271862#p1648048) if you're interested in the journey. The forum post - was thermally challenged and in order to find the best solution I needed to monitor the Pi.

Naturally, I used [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) passing data to [Influx DB](https://www.influxdata.com/products/influxdb/) and [Grafana](https://grafana.com/) deployed to a Kubernetes cluster to visualise the data (I'll share the configs in a future post). I have since moved to a micro-sized 3d-printed cluster with a single 92mm Noctua fan with _much_ better temps, but more on that in a future post.

One of the key decisions I made when setting up the cluster is that I wanted to run Ubuntu 20.04, and I wanted to run the 64bit userland. Unfortunately, unlike Raspbian, Ubuntu for Raspberry Pi does not come with the `vcgencmd` tool, nor is it available in the apt repositories, so I needed an alternative method;

## The reason we're here - vcgencmd

### Why?

If like I am, you're running Ubuntu on your Raspberry Pi's, you'll likely find, as I did, that the `vcgencmd` tool (which provides useful information such as whether the Pi is throttling, or has power issues as well as things like clock frequencies) is not available in the distro or the repo.

A quick search revealed that there is a PPA (Personal Package Archive) on Launchpad from an unofficial source; it didn't look very active, and honestly I prefer not to install or use random PPAs anyway, so on to option... well, the only option; build from source.

Building from source is extremely easy, the [Raspberry Pi Github repo](https://github.com/raspberrypi) contains the source for the userland tools, amongst other things, with a handy build script; all you need is a cross-compiler and a little knowledge to put things in the right place (I'll provide the latter here), so let's get started;

> NOTE: These instructions assume you are running a Linux distribution (Ubuntu or compatible) or your workstation, I may provide a Docker image to remove this requirement if time allows; for now, set up an Ubuntu 20.04 VM to follow along if you aren't running a suitable OS.

### Let's get building

Starting on your Ubuntu workstation or VM;

1. First, install the cross-compilers needed to build the code;
```console
sudo apt install -y gcc-aarch64-linux-gnu g++-aarch64-linux--gnu
```
2. Clone the userland from the official Pi repo;
```console
https://github.com/raspberrypi/userland.git
```
3. Change directory to the `userland` repo;
```console
cd userland
```
4. Run the build script;
```console
./buildme --aarch64
```
5. If you're been successful so far, the hard part is over, congratulations! If you haven't gotten this far, there could be a number of issues;
    * Check that the packages above installed correctly
    * Check you're on Ubuntu 20.04 x64
    * I used commit `093b30bbc2fd083d68cc3ee07e6e555c6e592d11` of the Raspberry Pi userland
    * I used version `9.3.0-1ubuntu2` for the gcc and g++ cross compiler packages.
    * Steps 1-4 should be carried out on your workstation/VM and not on the Raspberry Pi.
6. Now we're going to copy the binary and shared libraries to the Pi;
7. Copy `build/bin/vcgencmd` to the Raspberry Pi and move it to `/usr/local/bin/vcgencmd`;
```console
# Run this command on the Pi
sudo mv /path/to/vcgencmd /usr/local/bin/vcgencmd
```
8. SSH to your Pi and change permissions on the file;
```console
sudo chown root:video /usr/local/bin/vcgencmd
sudo chmod 775 /usr/local/bin/vcgencmd
```
9. Copy shared libraries needed by vcgencmd to the Pi, you will need `build/lib/libvchiq_arm.so` and `build/lib/libvcos.so`, then move them to the `/lib/` directory on the Pi;
```console
sudo mv /path/to/libvchiq_arm.so /path/to/libvcos.so /lib/
```
10. Make sure your user is a member of the `video` group;
```console
groups
```
11. If `video` is not listed, add it; substituting `<user>` for your Pi user;
```console
sudo usermod -aG video <user>
```
12. Exit the SSH session on the Pi and log in again to join your new group
13. Add a udev rule to set the permissions on the vchiq device;
```console
echo 'SUBSYSTEM=="vchiq", GROUP="video", MODE="0660"' | sudo tee /etc/udev/rules.d/99-input.rules
```
14. Reload the udev rules;
```console
sudo udevadm control --reload-rules && sudo udevadm trigger
```
15. Test `vcgencmd`;
```console
vcgencmd measure_temp
temp=39.0'C
```
16. You're done! Have a look at the [official documentation](https://www.raspberrypi.org/documentation/raspbian/applications/vcgencmd.md) for more commands you can run with `vcgencmd`.

## Future Work
There are of course ways this process could be improved, first, the build process could happen in a Docker container, writing a suitable Dockerfile should be trivial, however, the bulk of the work on your part is moving the relevant files and setting up permissions etc and the Dockerfile won't help with that, perhaps I could have the Docker container build a Debian package (.deb), seems like a useful thing to learn for 2021, so perhaps I'll add that in an update!

## Troubleshooting

### Permissions Denied or similar
Check that the vchiq device is correctly owned by root:video;
```console
ls -la /dev/vchiq
crw-rw---- 1 root video 236, 0 Jan  3 00:36 /dev/vchiq
```
If it is not, or the permissions are not `660` as above, check your udev rules, if it looks correct, try rebooting

Check that your user is a member of the `video` group by running the command `groups`, if not, log out/in or reboot; if it is still not, follow the command above to add your user to the video group (step 11).

### Shared Library Error
For shared library errors, such as `vcgencmd: error while loading shared libraries: libvchiq_arm.so: cannot open shared object file: No such file or directory`, check that both shared libraries were copied to the `/lib/` directory on the Pi and that their filenames are correct; `libvchiq_arm.so` and `libvcos.so`

For other issues, hit me up on [Twitter](https://twitter.com/0x2b2b) and I'll see if I can help!
