---
layout: post
title:  "zypper single transaction mode now default"
date:   2025-07-29 10:48:00 +0200
author: Thorsten Kukuk
---

## Default zypper backend

Over the weekend we switched the default zypper backend from the `classic_rpmtrans` to the `single_rpmtrans` backend in openSUSE MicroOS.

Anyone who has followed the factory mailing list or openSUSE's bugzilla over the last two years has probably noticed the regular discussions about faulty updates, mainly because libraries were updated too late. The result was that tools from %pre/%post scripts did not work and in the worst case made the system unbootable.
Traditionally, zypper executes the rpm command separately for each operation in a transaction. The advantage is that with large updates and a small root partition, zypper can download the packages one after the other, making the update possible in the first place.
One of the disadvantages, however, is that the update process is much slower with a large number of packages. And not only that, zypper also has to save the posttrans scripts and execute them manually at the end. This means that the update process runs differently than when all RPMs are updated in one with RPM.
Therefore, the zypper developers have implemented a new backend that executes all operations in a single transaction with librpm. However, for various reasons this is not yet the standard and so there are still bug reports about faulty bootloader configurations or problems updating libopenssl when using hardware-optimized libraries.


## How is this implemented?

On the command line and for systemd units, the environment variable ZYPP_SINGLE_RPMTRANS=1 is now set globally, which should cover most cases on MicroOS.

If there are still situations where zypper uses the old classic backend, please report this via bugzilla.

## What does this mean for the users?

Hopefully: **nothing**

Or better: the update problems are gone. If you still see problems with packages, please report them via bugzilla so that we can fix them.
