---
layout: post
title:  "Switch from wtmp to Y2038 safe wtmpdb"
date:   2023-06-28 10:48:00 +0200
author: Shawn W Dunn
---

## Intro

With the last updates, openSUSE MicroOS, Tumbleweed and variants saw a new package: [wtmpdb](https://github.com/thkukuk/wtmpdb)

This is a replacement for wtmp.

## Y2038

**Why do we need a replacement for wtmp? wtmp is really old and well accepted?**

On January, 19th 2038 at 03:14:07 UTC the 32bit time_t counter will overflow. The general statement so far has always been that on 64bit systems with a 64bit time_t you are safe with respect to the Y2038 problem. But glibc uses for compatibility with 32bit userland applications 32bit time_t in some places even on 64bit systems. This is the case for `struct utmp` used by wtmp.

Changing the `struct utmp` in glibc to use a 64bit time_t is pretty complicated and in every case ABI incompatible, beside that the on disk format will change incompatible, too. For this reasons, the glibc developers plan to deprecate the API at some point in the future.

For this and other reasons, we developed `wtmpdb`, which collects all data via the PAM module `pam_wtmpdb.so` and where `wtmpdb last` provides a compatible replacement of `last`.

`wtmpdb last` should work already today and show you similar output
compared with `last` itself. The next step will be to make
`last` a link to `wtmpdb last` and rename the old last binary to
`last.legacy`.

There are currently no plans to disable writing of wtmp entries
completly, this will most likely happen together with utmp.
Applications reading wtmp directly should continue to work, but such
applications are really rare. Applications writing to the wtmp file should
not create any problems.


## What does this mean for the users?

Hopefully: **nothing**

`wtmpdb` got already introduced in our codebase and will be installed by default. For older systems, updates should pull it in automatically and the systemd units should get automatically enabled.
If the package `wtmpdb` is not installed on your systems, you can install it with:

* On MicroOS and variants: `transactional-update in wtmpdb` and reboot to activate it
* On Tumbleweed: `zypper in wtmpdb`

## Important

Due to a about one year old bug in `systemd-presets-common-SUSE`, the wtmpdb unit files `wtmpdb-update-boot.service` and `wtmpdb-rotate.timer` didn't got always enabled as they should be. If you haven't made a fresh installation, please enable them yourself:

* `systemctl enable --now wtmpdb-update-boot.service`
* `systemctl enable wtmpdb-rotate.timer`
