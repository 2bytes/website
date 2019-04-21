+++
title = "SSH 2-Factor on Linux (RPi)"
date = 2019-04-07T11:25:10+01:00
draft = false
+++

In this post I will show how simple it is to enable 2-Factor authentication using a Raspberry Pi, and your smartphone,
but this can also be done for other Linux devices and Servers too.

<!--more-->

## Why 2-Factor?

With people's information being leaked left, right and centre, it's increasingly more critical that we rely on more than
just our passwords for access to important devices. If we do not, a leaked password could end up with our servers and devices, such as Raspberry Pi's
being hijacked to participate in a bot-net or some other nefarious deed, or information on those devices being stolen.

## First Steps for Raspberry Pi users

Before beginning the setup of 2-Factor auth on your Raspberry Pi, there are some good-practice steps you should carry out first.

> If you're not running a Raspberry Pi, but some other linux machine, such as Ubuntu, Debian, Linux Mint etc, feel free to skip this step. 

1. Change the password for the `pi` user (default is `raspberry`), make it really strong, you shouldn't need to use it after adding your own user (I prefer to delete the user altogether).
    - Enter the command `passwd` while logged in as the `pi` user, you'll be prompted to enter the existing password, then the new one, twice.
2. Create a new user, other than `pi`, with your own name, for example, I would use `hamid`, with a password that is strong but memorable.

```shell
sudo adduser hamid
Adding user `hamid' ...
Adding new group `hamid' (1001) ...
Adding new user `hamid' (1001) with group `hamid' ...
Creating home directory `/home/hamid' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hamid
Enter the new value, or press ENTER for the default
    Full Name []: Hamid
    Room Number []: 
    Work Phone []: 
    Home Phone []: 
    Other []: 
Is the information correct? [Y/n] y
```

4. Add your user to the `sudo` group so you can perform admin tasks with the `sudo` command.

```shell
sudo usermod -aG sudo <username>
```
The `pi` user is a member of lots of other groups, your new user will start as a member of only your own name group.

Be sure to add other groups if you need access to other features, such as the `gpio`, `i2c` or `spi` on your Pi for your projects.

```shell
groups pi
pi : pi adm dialout cdrom sudo audio video plugdev games users input netdev spi i2c gpio
```

Now, in a new terminal/ssh client you should be able to ssh into your Pi as your new user, this is the user you should access the Pi with from now on.

## Setting up your own 2-Factor

> This post assumes a debian based Linux distro, and therefore the `apt` package manager, mainly because the Pi runs Raspbian, based on debian.
> If you use another distro on your server, you can still follow this, but installing packages will use a different package manager and possibly package names.
> You might even use a different editor instead of `nano` such as `vim` or `emacs`.

You'll need a smartphone/tablet, and the FreeOTP app by Redhat in order to generate the one time codes you'll use on login. This is available on Google Play and the App Store.

First start by logging in to your Raspberry Pi, you can do it by SSH, but make sure to keep a spare terminal open while you're 
changing the SSH configuration, as it will remain connected even if you break the ssh configuration, or connect a monitor, keyboard and mouse and make changes directly on the Pi.

Be sure to test your connection using a second terminal, leaving the first one for emergencies.

First login to the Pi using your new user (if you're using a keyboard/mouse directly attached, just login normally).

> I'll use the user `user` you should substitute your own. If `raspberrypi` doesn't work, you'll need the IP address of your Pi after the @.

```shell
ssh user@raspberrypi
```
If configured correctly you user should have `sudo` access (see above; you need to log out and in again after adding your user to a group).

First we need to install the pam-oath plugin, this allows the authentication system to support [oath](http://www.nongnu.org/oath-toolkit/index.html)
and the qrencode tool to add the one time pass to our smartphone app.
```shell
sudo apt-get install -y libpam-oath qrencode
```

Next, edit the sshd config to enable challenge-response authentication:
```shell
sudo nano /etc/ssh/sshd_config
```
Set the following line from `no` to yes
```shell
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication yes
```

Edit the sshd pam config:
```shell
nano /etc/pam.d/sshd
```

Add a new line, near the top of the file, under `@include common-auth`

```shell
auth requisite pam_oath.so usersfile=/etc/users.oath digits=6 window=30
```
This tells pam that we want to add an auth requirement, that it should use the pam_auth plugin, 
the users can be looked up in `/etc/users.oath`, that our passes will be 6 digits and valid for 20 seconds.

Now we need to generate our secret and set it in the `/etc/users.oath`, you can use `openssl` to do this:

```shell
openssl rand -hex 20
```

Next, lets add it to the file:
```shell
sudo nano /etc/users.oath
```
The file should be empty if this is your first user, enter the following, pressing `Tab` rather than `space` between each block.
```shell
HOTP/T30/6  user    -   <paste your secret from above here>
```
It should look something like this:
```shell
HOTP/T30/6  user    -   f21cffb7e5459896b298e65d586117186cd3bb15
```

Install the "FreeOTP Authenticator" from RedHat" on your Android or iOS phone, and generate the QR code for it to scan.

First, we need to convert the secret from above, to Base32, which is required by the QR app:
```shell
echo <your secret> | xxd -r -p | base32
```
The output will be a Base32 string, something like this:
```shell
6IOP7N7FIWMJNMUY4ZOVQYIXDBWNHOYV
```

Now generate a QR code to scan with your phone:

```shell
qrencode -t UTF8 "otpauth://totp/RaspberryPi%20SSH:user%40raspberrypi?secret=<yoursecret>&issuer=user"
```
Be sure to replace `user` in both places with your user, and put your secret Base32 from the last step just after `secret=` and before `&issuer`.

When you run this, `qrencode` will print out a QR in your terminal, press the QR icon in the App on your phone and scan it,
if successful you should see a new item added to your codes list in the app.

If it fails to scan, make sure you didn't accidently remove any characters above, make sure you used the Base32 secret, and try generate the QR code again.

Finally, we need to restart the ssh server, and test our login;
```shell
sudo service ssh restart
```
Open a new terminal (don't close the old one, you might need it if anything is wrong)
```shell
ssh user@raspberrypi
```
You'll be prompted for your password:
```shell
Password:
```
Once you type that correctly, you should be prompted for your One-time password:
```shell
One-time password (OATH) for `user': 
```
Tap the entry in your mobile App and it'll display a 6 digit pass valid for 30 seconds or less, enter that and you'll be logged in!

If you can't log in, double check all of the previous steps, and if you change any configuration files, be sure to restart ssh `sudo service ssh restart`.

That's it, now you have secured your Pi with 2-Factor authentication, from now on, you'll need your phone and the FreeOTP app to login.

## Cleanup

Final note. It is good practice to clear your shell history since the secrets we've used above will be in there.

You can type `history` to see what I mean.

Run the following to clear the entire history
```shell
history -c
```
This will remove all entries from your shell history on the Pi (or the machine you ran it on, if not the Pi).

Or to remove just a single entry by number:
```shell
history -d 24
```

Better practice would be to not enter secrets on the command line, but use files, or linux pipes to direct them straight to their destination, 
but that is a more advanced topic for another day.
