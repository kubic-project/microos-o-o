---
author: Fabian Vogt
date: 2024-05-17 11:20:37+02:00
layout: post
license: CC-BY-SA-3.0
title: Easily set up Passwordless SSH Authentication
categories:
- Announcements
- openSUSE
tags:
- openSUSE
- Community
- Security
- MicroOS
- Tumbleweed
- Rolling Release
---

MicroOS and Tumbleweed gained a new feature: You can now easily enroll SSH keys for authentication through the firstboot wizard on the very first boot already!

# Motivation

Public-key based authentication is safer than password authentication and in most cases also easier to use and maintain. By default, OpenSSH does not even allow root to authenticate with a password anymore.

Question is how to get the desired public keys into the system to be able to log in. ssh-copy-id needs to be able to log into the target system, which requires password auth. Transferring the keys manually is tedious and error-prone.

With [ssh-pairing](https://github.com/Vogtinator/ssh-pairing) there's a new approach: a dialog based workflow allows interactive key exchange in both directions. After making an ssh connection and accepting the enrollment, the server's key is added to the `known_hosts` lists on the client and the selected public keys from the client added to the server's `authorized_keys` file.

# How does it look like?

After starting the pairing procedure, the dialog shows what to do:

![ssh-pairing showing connection info](/assets/images/ssh-pairing/connection-dialog.png)

After the connection has completed, keys to accept can be individually reviewed and selected.

![ssh-pairing asking whether to import a key](/assets/images/ssh-pairing/key-import-dialog.png)

# Where can I get it?

Starting from Snapshot 20240507, MicroOS and Tumbleweed images include this feature in the firstboot wizard. If you want to make use of this on other systems, you can install the `ssh-pairing` package and run the command with the same name (as root).

# What about other distros?

ssh-pairing is distro-independent, see the [project page](https://github.com/Vogtinator/ssh-pairing?tab=readme-ov-file) for installation and usage instructions.

It would be very welcome to have this available for other distros out of the box as well! If you have questions or find any issues, please don't hestitate to open tickets.

Have a lot of fun!
