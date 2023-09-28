---
layout: post
title:  "Y2038: Replace utmp with logind"
date:   2023-09-28 08:34:00 +0200
author: Thorsten Kukuk
---

## Intro

With the last updates, many packages of openSUSE MicroOS, Tumbleweed and variants switched from `utmp` to `systemd-logind`.

## Y2038

**Why do we need a replacement for utmp? utmp is really old and well accepted?**

On January, 19th 2038 at 03:14:07 UTC the 32bit time_t counter will overflow. The general statement so far has always been that on 64bit systems with a 64bit time_t you are safe with respect to the Y2038 problem. But glibc uses for compatibility with 32bit userland applications 32bit time_t in some places even on 64bit systems. This is the case for `struct utmp` used by wtmp.

Changing the `struct utmp` in glibc to use a 64bit time_t is pretty complicated and in every case ABI incompatible, beside that the on disk format will change incompatible, too. For this reasons, the glibc developers plan to deprecate the API at some point in the future.

Beside the Y2038 problem, the utmp API has several additional problems:

* There are design issues which allow a DoS attack [utmp/wtmp locking allows non-privileged user to deny service](https://sourceware.org/bugzilla/show_bug.cgi?id=24492)
* Due to the fixed format the length of the username and device name is limited to 32 characters, while nowadays all other tools allow much longer or even unlimited names. And strings are null terminated unless they use the full length of the variable. This complicates the parsing of the data and leads to errors in the applications every now and then.
* The usage, especially who should create which entry, is not defined. As result: if you use GNOME and start 5 terminals (gnome-terminal), `/usr/bin/who` will report one user. If you use KDE or xdm and start 5 terminals (konsole or xterm), `/usr/bin/who` will report six users. Same if you use screen or tmux: with screen `/usr/bin/who` will report every session an extra user, with tmux you will only see one user. So you can only use the data for informative reasons, but you cannot trust them for monitoring or something similar.

After evaluating several different alternatives, the decission was made to use `systemd-logind` instead:
* systemd-logind is always running
* there is one central tool controlling, what a session is and what not
* the API with libsystemd does not suffer from any of the utmp API problems

## What does this mean for the users?

Hopefully: **nothing**

All big projects accepted our patches or wrote their own support for this, and the majority of them did already release a new version. This are e.g. coreutils, procps, shadow, util-linux and systemd itself.
Using `who`, `w` or similar tools should give you a similar output as before, but there is one big difference: you will no longer see xterm, konsole, screen or similar sessions in the output.

## Outlook

For compatibilty reasons `/run/utmp` will still be created until the majority of packages are adjusted. It's expected that latest with systemd v255 (which contains the necessary patches for broadcast messages without utmp) `/run/utmp` will no longer be created.

* Some packages are still missing:
  * qemu: patch is WiP
  * samba: patch exists and is tested, integration missing
  * net-snmp: SR pending
  * rsyslog: SR pending

## Presentations

* [Recording of All Systems Go!](https://media.ccc.de/v/all-systems-go-2023-183-y2038-replace-utmp-with-logind)
* [OSS Europe 2023](https://osseu2023.sched.com/event/1OGgG/y2038-and-utmpwtmp-on-64bit-systems-thorsten-kukuk-suse)
