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

The `/etc/sysconfig/network/ifcfg-*` files should be compatible, but it's not
clear if there are no corner cases where they are incompatible. A migratoin
should be pretty easy in this case. But if a native wicked xml configuration
is in use, a manual migration of the configuration has to be done.

## Migration

If the network configuration got created during installation or if only
`ifcfg-*` files are used, the switch to NetworkManager should be very
simple. Just replace wicked with NetworkManager:

```
# transactional-update shell
-> zypper in --no-recommends NetworkManager
-> rpm -e wicked wicked-service
-> systemctl enable --now NetworkManager
-> exit
# reboot
```
