---
title: Using a U2F key with a mac
layout: post
permalink: using-a-u2f-key-with-a-mac
published: true
---
There are tutorials explaining how to configure OTP Yubikey devices to work with mac login, but what about those of us who only have U2F keys? It is possible! but more complicated...


### What you need
* Your mac (I have a mid 2012 Macbook Pro)
* A U2F key (I use https://www.yubico.com/products/yubikey-hardware/fido-u2f-security-key/)

### Setting up brew
You will need to have Brew installed. If you don't yet have brew: enter the following in a terminal prompt.

```sh
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### Installing U2F libraries
Thanks to some awesome people who ported the u2f libraries to mac, you won't need to compile them yourself. 

```sh
$ brew install pam-u2f
```

### Copy the library into place
This will allow us to use the library in the mac PAM config files.
```sh 
$ sudo cp /usr/local/Cellar/pam-u2f/1.0.4/lib/pam/pam_u2f.so /usr/lib/pam/pam_u2f.so
```

### Save a mapping file
Run the following command to get your key ID
```sh
$ pamu2fcfg -u $(whoami)
```
If the key has a button, tap it. Copy the result and place it in a file (`/etc/u2f_mappings`).

### Configure the library with PAM

==**STOP!**== You will need to disable "System Integrity Protection", to configure this. If you have it enabled:

* Restart your computer and hold COMMAND+R. 
* Then click "Utilities" and "Terminal". 
* Enter "csrutil disable" and press enter. 
* Then reboot.

This is a bit tricky because screwing something up means you won't be able to access your computer. So we will start by adding the key to something to is easily reverted, the screensaver.

```sh
$ sudo nano /etc/pam.d/screensaver
```
and add the line

```
auth required pam_u2f.so authfile=/etc/u2f_mappings
```
Now you will want to test it out. You can do this by modifying your security settings to require the password immediately.
![](https://blog.avisi.nl/wp-content/uploads/2014/03/screensaver-300x243.jpg)

Enable your screensaver! Try to enter your password without the key and it should fail. You will have to enter your password and then tap your key to unlock your computer. If you can't get into your computer, click "switch user" and login that way, you will be able to go back and remove the line you added.

### And more files to edit
You can now edit the following PAM files to finish setting up your key, add the same line as you did above

* `/etc/pam.d/sudo` - Require key when running sudo commands
* `/etc/pam.d/authorization` - Login and authorization dialogs

### Reboot!
Reboot for good measure. If you disabled System Integrity Protection, now would be a good time to enable it again. You can do that by going into recovery mode, opening the terminal, and running `csrutil enable`. 