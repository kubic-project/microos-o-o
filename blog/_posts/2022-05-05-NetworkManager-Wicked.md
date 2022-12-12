---
layout: post
title:  "Switching from wicked to NetworkManager"
date:   2022-05-05 16:10:00 +0100
author: Thorsten Kukuk
---

## Intro

NetworkManager was used already a long time for the majority of openSUSE
Tumbleweed installations, except for server, MicroOS and Kubic.
But more and more users requested NetworkManager also for this flavours, since
wicked is missing some functionality (like 5G modem support) or there are k8s
network plugins, which only support NetworkManager. And since MicroOS Desktop
was already using Networking, it was a logical choice to switch completly.
So openSUSE MicroOS and openSUSE Kubic are now using NetworkManager as default
instead of wicked since quite some time.


## Configuration files

As of today there are no plans to automatically switch a system from wicked to
NetworkManager. The reason is: depending on the configuration, it may be as
easy as just replacing wicked with NetworkManager and everything will continue
to work, or, in worst case, everything needs to be re-created from scratch for
NetworkManager. There is no feature parity between this tools, so an automatic
conversation may not work in all cases.

And even worse, some tools store configuration data in different places
depending on the network stack. E.g. firewalld zone informations are either
stored in the NetworkManager configuration files, in ifcfg-* or in the native
firewalld configuration files.

## Migration

If the machine get's configured via dhcp and no fancy network tools or
configuration is used, exchanging wicked with NetworkManager is quite simple:

```
# transactional-update shell
-> zypper in --no-recommends NetworkManager
-> rpm -e wicked wicked-service
-> systemctl enable --now NetworkManager
-> exit
# reboot
```

But if the network configuration is not a simple dhcp client or if there are
problems with the network afterwards, the clear recommendation is to make a
fresh installation of openSUSE MicroOS. With the openSUSE MicroOS design, this
should be quite simple and fast.
