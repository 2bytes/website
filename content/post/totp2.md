---
title: "SSH 2-Factor's First Factor"
date: 2019-04-28T12:42:11+01:00
draft: true
---

In my previous post ["SSH 2-Factor on Linux (RPi)"]({{< ref "/post/totp.md" >}}) we enabled OTP as a second factor. That's a nice step in securing your SSH sessions however it's not the only step,
and in fact is usually not the first.

In this post I'm going to explain a little more about the first factor of two factor authentication for SSH and give a tutorial on using cryptographic key pairs 
for your first factor instead of using a plain old password.

As always, I try to be as beginner-friendly as possible so if any of the following is obvious, please skip ahead to the [tutorial]({{< relref "/post/totp2#tutorial" >}}).

<!--more-->

# What is a factor anyway?
When we talk about "factors" in authentication, they are essentially how many "things" we use as part of a single authentication; I say "things" because it's not always something you know,
it can, and often is a combination of something you know, a password, and something you have, a phone or hardware OTP (One Time Pass) generator.

## Analogy Time

Lets say I want to enter a special club house, and this special club requires a secret password. The doorman will ask me for the secret password.
If I know the password to the club, I can tell the doorman it, and enter. That was a single-factor authentication. I used one thing I know to gain entry.

Already, the club is more secure because I can only enter with the password. But is that enough? If it is, great, leave it at that and we're done. But what additional risks are there?

What's to prevent me from giving the secret door code to my best friend, or for some villainous rival from the club down the road overhearing and trying to get in?

With single-factor authentication, the doorman only cares that you know the password, but he doesn't know or care whether you belong in the club.

That's where second-factor comes in. If the club issues ID cards to its members, not only can I prove that I know the password (something I know), but I can also prove I am a member
of the club using my ID card (something I have). 

The astute among you will immediately realise that although better, this is still not perfect. It doesn't prevent me from giving the ID card to my friend,
but it does prevent the rival from the club down the road from gaining entry just by overhearing the password, unless of course he steals the ID card from me.

The way we solve this is by taking extra precautions to protect those factors. By employing a combination of something we have and something we know, we add further protection
because it's difficult to compromise these things using the same method.

# What factors are there?
Digitally, authentication factors can come in many forms such as passwords, cryptographic keys, One-Time-Pass generators, Smart Cards, Biometrics (fingerprints, voice, iris) and many more.

The "virtual" types commonly come in the form of a digital file, or string of characters.

One-time-pass generators can be found in the form of a mobile application, or in a physical device such as the little Red and Black "Secure Key" that HSBC bank gives out, 
or the Blue and White card reader that Barclays bank does, or even an RSA SecurID one might use at work.

Other devices in the form of a Smart Card (such as a chip-and-pin bank card) or a USB device like the FIDO "dongles" or a Yubikey, or a fingerprint reader are also all forms that such devices can be found in.

# Which factors should I use?
The types of factor used depends on what's called the "threat model". What kind of security threat does one envision to their system? Think of it like the club house, 
and what kind of device would mitigate the risks, and who the adversaries we're worried about are and what their capability is.

* If a password is practical to remember, and you always have a phone on your person, which you can install an app for TOTP (Time-based One-Time Pass), then go with that.
* If a password is deemed not strong enough, a cryptographic key-pair could be used as the first factor, and TOTP for the second.
* If a cryptographic key-pair is a good first factor, but a phone is not ideal for second-factor, a smart card type solution or hardware OTP generator can be the second.
* Many more combinations exist, and the number of factors used is not limited to 2 (hint: during this tutorial you'll end up with 3-factors at one point).

The guide in this post will go with the second option, because it is practical, and yet more secure than the basic Password/OTP combination from the original post, 
with an added caveat that you must have access to the cryptographic key's private part in order to log in using this method.

# Which factors should I NOT use?
A somewhat popular second-factor of late is sending an OTP to a mobile device by SMS, some banks such as Santander do this, as do lots of websites and other institutions.

Due to the recent popularity of what is know as a "SIM swap scam" I would highly recommend against this method if it can be avoided.

The way SIM swap scams work is that an adversary will contact a victim's mobile phone provider and use social engineering, bribery or other means to convince the provider's representatives
to "switch" the number (your phone number) to another SIM which will end up in the adversary's possession. At this point, they essentially own your phone number (you'll lose access to it) and they can receive
any calls and messages that were intended for you, such as a one-time-pass or other SMS or Phone based confirmation. By the time you realise and/or are able to contact your provider about the issue, they could have broken
into all sorts of things.

I recommend searching for stories about this growing concern, online, understanding the risks, and where possible, ask questions to those using this method of authentication so that
hopefully we can encourage the use of such authentication methods to stop.

# <a name="tutorial"></a>Enough, just give me the tutorial already!

This tutorial is intended to be a follow up to the first, ["SSH 2-Factor on Linux (RPi)"]({{< ref "/post/totp.md" >}}), and should be used alongside it.

As always, this post will cover a specific point and will not cover all options and configurations that can be set to harden SSH, but with default settings combined with this tutorial
and the previous, you'll already be much safer than just a single password login alone.

For a server or machine that you consider particularly important, I recommend looking up "SSH hardening" and configuring, with caution, your SSH server for tighter security.

### Warning - Before you begin

Before you begin, ensure that you have an alternative means of accessing the machine in question. If you're using a Raspberry Pi, ensure you're able to connect a keyboard/mouse and monitor.
If you're using a Virtual Private Server in the cloud, make sure that your provider offers console access and you're able to access it.

> _If you lose your SSH private key you will no longer be able to SSH into your machine_ and will be forced to login by other means to install a new key (and remove the one you've lost).

### 0 - Preparation

If you followed the previous tutorial, you already have 2-factor authentication. Open an SSH session to the machine you're doing this for
and leave it aside. If you break the login or mix up the keys, this will be the easiest way to get back in and fix it.

###  1 - Keys

In order to replace password login with SSH keys, we first need to generate a public/private key pair.

> Note: This key pair should be generated on the machine you'll login FROM, i.e. your laptop or desktop. The private key should never leave this machine unless it is encrypted or protected by some other means first.

During generation you'll most likely be prompted to give the private key a password to protect it. I <s>highly recommend</s>, in fact, I insist, set one. It should be different to the password you normally login with for extra security.

> ProTip: If you use multiple machines to login to the SSH server, you should generate a new key pair for each. If one is compromised, you only have to replace it, and still retain access via other machines.

I'll explain how to generate your keys on Linux, because I use Linux, but, if you use other OSs there are ways to do so on each of those platforms (e.g. on Windows, you can use PuTTYGen).

> In this tutorial we'll use `RSA 2048`, because it is the default and generally well supported. Feel free to read up on alternatives such as `ECDSA` and `ED25519` and decide if you'd prefer those, or even a longer RSA key.

#### 1.0 - Public & Private

**Whichever platform you use, the important thing to remember is that SSH keys come in pairs; a Public Key and a Private Key.
The Public Key goes on the machine you want to log in TO. The Private Key goes on the machine you want to login FROM.**

Read that again, and make sure it's clear. The *Private Key* never leaves the machine you're sitting at. It's *private*.

#### 1.1 - Generating your key pair on Linux

When you login to a Linux system from another, the process and generating and adding keys is fairly streamlined.

Although I cannot explain how to generate key pairs on other platforms I will try to explain how and where they are placed so that if you're figuring this out for your platform, it should be less painful.

Run the SSH keygen tool. You'll see that the default location for placing your private key `id_rsa` is in a hidden directory `.ssh` inside your home directory.
```shell
ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):
```
Although you can change this location, I suggest leaving it as-is for now and simply hitting ENTER. You'll be prompted to set a passphrase.
```shell
Enter passphrase (empty for no passphrase):
```
This passphrase is used to "unlock" the key so that it can be used to login. If you use a desktop OS like Ubuntu or Linux Mint, you can configure it to unlock this for you automatically when you login
to your machine. You are allowed to leave it blank, but I discourage doing so. If anyone gets their hands on it, it can be used without an unlock passphrase.

> Note: This passphrase does not need to match any other, it is used specifically to unlock this key.

You'll be prompted to enter your passphrase a second time to confirm it, then your keys will be generated and placed as follows;
```shell
Enter same passphrase again: 
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:T1PrwZNV576pYcV6E+yOMGB2wTKtnNiM/Zi4zMdyo70 user@Desktop
The key's randomart image is:
+---[RSA 2048]----+
|                +|
|               o+|
|            ..==*|
|           o.*=X+|
|        S oo*o+=o|
|         o +==O.+|
|          . =X.B |
|            ooB .|
|             oEo |
+----[SHA256]-----+
```

> Fun Fact: The randomart image is a human-friendly way you can visually compare fingerprints without reading long strings. It's not currently in wide use (or enabled by default).

#### 1.2 Installing your Public Key to the remote machine

Now we have our key pair we can install our Public Key to the remote machine. Think of this process like giving the remote machine a snapshot of your ID card, so that when you connect, it can identify you.

In cryptography this process is a little more complicated than the club house analogy, because you give the doorman your Public Key, you never actually reveal your secret key or "secret password".

The process of installing you Public Key however is not complicated. First I'll explain how to do it, in a "bonus" section at the end I'll explain what is happening for those interested, and how to perform this step manually if desired.

Install your public key to your remote machine; Here we'll use the Raspberry Pi from the first tutorial:

> Note: If you used the IP address to access the Pi in the previous tutorial, do the same here

We can use the `ssh-copy-id` command to install the Public Key to the correct place on the remote machine, we just login using our existing login methods.

> WARNING: Be sure to select the `.pub` file!

```shell
ssh-copy-id -i ~/.ssh/id_rsa.pub user@raspberrypi
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
Password:
```
Enter the password you use to login to the Raspberry Pi or server. If you followed the first tutorial, you'll also be asked to enter your OTP.

```shell
One-time password (OATH) for `user':

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'user@raspberrypi'"
and check to make sure that only the key(s) you wanted were added.
```

Now you can try logging in with your new key, on Linux it's really simple, just login with username and server address (this may be the IP instead of the host name).
```shell
ssh user@raspberrypi
```
If you're on the likes of Ubuntu, you may get a dialog pop up that asks you to unlock your key. Use the passphrase you set earlier when generating the key.

In this dialog you can also choose to unlock the key automatically on login, this is fine if your laptop/desktop is secured adequately with a decent password.

You should immediately be logged in to your Pi or server without being prompted for your password, or your OTP for that matter.

By using keys, we sidestepped our previous password/OTP requirement and logged in with just our keys.

Now we need to configure SSH properly on the Pi (or server) to ensure we're prompted for our OTP again and finally to disable the password requirement (if we wish).

### 2 - Configure SSHd

First, ensure you still have your second SSH session open and working, keep it to one side.

Next, in your newly logged in session, from the previous step, edit the SSH config file:

```shell
sudo nano /etc/ssh/sshd_config
```
In the file, add a new line under the `UsePAM:yes` line:
```shell
AuthenticationMethods publickey,keyboard-interactive:pam
```
This tells SSHd which authentication methods are required and in which order, so here, we are saying we want `publickey` first, followed by `keyboard-interactive:pam`, the second part
is our password/otp combination we previously configured.

You should also disable password authentication and root login for additional security (Remove # at start to activate a configuration otherwise it will be ignored):

These options will not necessarily be next to each other in the file so look around for them.
```shell
PermitRootLogin no
PasswordAuthentication no
```

If you restart SSHd now, you'll have 3-factor authentication (Woohoo, it's even stronger than 2-factor)! You can restart SSHd and leave it there if you wish, 
and now you'll be required to provide your Public Key, Password and OTP in order to login.

```
sudo service ssh restart
```

If you don't want to enter your password each time you login with your Public Key, and just want to provide the OTP, we can remove the requirement for a password:

Edit the following file:
```
sudo nano /etc/pam.d/sshd
```
Comment out the line for common-auth by putting a `#` in front:
```
#@include common-auth
```
Restart SSHd
```
sudo service ssh restart
```

That's it, log out, log in, and you'll only be prompted for your OTP.

Remember, you can only login from a machine that has a public key installed to the server you're logging in to, that matched its private key. Follow 1.1 and 1.2 again to generate and install a key pair for another machine you use such as your desktop and laptop.

### Optional Bonus Reading - Installing Public Keys manually

When you run `ssh-copy-id` the command takes the login details you normally use, and installs the key you pointed it at into a specific place on the Raspberry Pi or server:

```shell
/home/user/.ssh/authorized_keys
```
This file list public keys, one per line that match a private key which is allowed to login here. An example will look something like this:
```
cat ~/.ssh/authorized-keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFKj8p5gv1g9uLRln2uSyvYXvmOYTokQ6eyFj8p/IgcFCzeruA8fS9DPJmqok74VpLkSl9lc7hVTvKLseOYpd+W5dgUoJeD+9tsmfAuJvGzINeKNpjAGlp+8NhySyMxBvPLvmtu4WqaG82G1R3bcVL/RiGWhiwjHPpnebNTvbgvKQKgvWNXaKMHyT974MfBjH0qiFdMAulbHUF+6npi/ieU+Tjo8+M/CdzWV0qO1iX1bNIJi2V+3A7Xl/KmcwWgjJk8u97vxeg7eYxPEnnjdO0UEsR7GLYTNuqN6XR6J9cImeLoZutCeChYQoK/nqXPA9H/aDRe8/nGilqDX5NDt53 user@Desktop
```
Each line will show the key-type `ssh-rsa`, other key types are possible. A space follows the key type, then the content of the Public Key, followed by a further space and a name to describe its purpose, in this case,
this is the key I "user" use to login from my "Desktop".

You can add public keys here manually if you're unable to use `ssh-copy-id`, simply open the file, go to a new line, and paste the entire content of the `id_rsa.pub` you generated earlier.

If the `authorised_keys` file does not exist, create it and set its permissions as follows

> run this as your own user, not root or with sudo

```
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 user:user ~/.ssh/authorized_keys
```
Be sure to substitute "user" in both places with your actual user name. Now edit the file, for example with `nano` and paste your public key in, making sure to press ENTER after to create a new line.

> ProTip: `~/` is short for `/home/user/` where `user` is automatically filled in using your current user.