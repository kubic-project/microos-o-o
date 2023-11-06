---
layout: post
title:  "Y2038: /run/utmp, /var/log/wtmp and /var/log/lastlog are gone"
date:   2023-11-06 15:41:00 +0200
author: Thorsten Kukuk
---

## Intro

With the last updates openSUSE MicroOS and Tumbleweed don't use `/run/utmp`, `/var/log/wtmp` and `/var/log/lastlog` anymore. This files will no longer be created. `/run/utmp` will vanish with the next reboot, `/var/log/wtmp` and `/var/log/lastlog` will not be created with a fresh installation. On existing systems, the files will stay, but we will not read it anymore. It can be safely moved away by the administrator.

## Background

As already explained in the last two blogs [Y2038: Replace utmp with logind](../2023-09-28-replace-utmp-with-logind/) and [Switch from wtmp to Y2038 safe wtmpdb](../2023-06-28-switch-to-wtmpdb/), this files are not Y2038 safe. `/var/log/wtmp` got replaced with `wtmpdb`, `/run/utmp` with `systemd-logind` and `/var/log/lastlog` with `lastlog2`.

## What does this mean for the users?

Hopefully: **nothing**

All big projects accepted our patches or wrote their own support for this, and the majority of them did already release a new version. This are e.g. coreutils, procps, shadow, util-linux and systemd itself.
Using `who`, `w`, `last`, `lastlog` or similar tools should give you a similar output as before, but there is one big difference: you will no longer see xterm, konsole, screen or similar sessions in the output.

## Presentations

There are two presentations explaining this in detail:

* [Recording of All Systems Go!](https://media.ccc.de/v/all-systems-go-2023-183-y2038-replace-utmp-with-logind)
* [OSS Europe 2023](https://osseu2023.sched.com/event/1OGgG/y2038-and-utmpwtmp-on-64bit-systems-thorsten-kukuk-suse)
