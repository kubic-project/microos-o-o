---
layout: post
title:  "Introduction of soft-reboot"
date:   2024-06-13 14:55:00 +0200
author: Thorsten Kukuk
---

## Intro

systemd v254 introduced a new reboot method: `systemctl soft-reboot`.
A soft-reboot is similar to a regular reboot,  except it affects user-space only. This command does:
* Shutdown of all running services and other units
* Optional: switch to a new root file-system
* Re-exec systemd
* Start all systemd services again

The advantage of this method is a pretty fast reboot:
* No hardware, firmware and bootloader
* No kernel
* No initrd

But there are some challenges to be solved:
* Kernel tables will not be flushed (may break services like firewalld+podman).
* No initrd means everything done in the initrd normally gets not executed. In the case of MicroOS, this will break mount of overlayfs for /etc with transactional-update or relabeling the system for a new SELinux policy.
* /run will not be cleared, which means reboot needed trigger will not be removed. This may affect many more services not expecting that files survive in /run.

Nevertheless, this method is ideal for quick reboots with transactional-update, e.g. if you only installed another package.

## Current status

Support for soft-reboot is fully integrated in transactional-update and rebootmgr. The known, reproducible problems with soft-reboot are solved.

Currently soft-reboot is still disabled in transactional-update, but will be enabled by default in the near future.

## How to use

* An admin can trigger an immediate soft-reboot with: `systemctl soft-reboot`
* A scheduled soft-reboot via rebootmgr can be triggered with: `rebootmgrctl soft-reboot`
* With `transactional-update` it is not possible to enforce a soft-reboot, if soft-reboot is enabled, `tranactional-update reboot` will choose the best fitting reboot method based on the list of package updates. See [man zypp-boot-plugin](https://manpages.opensuse.org/zypp-boot-plugin) for more details and how to configure this.

If a soft or a hard reboot was done can be checked with the `last` command. The entries `s-reboot` or `soft-reboot` indicates a soft-reboot was done, `reboot` means this was a normal reboot. The detection will only work reliable with systemd v256 or newer.

## Installation and Configuration

A prerequisite for this feature is that the `patterns-microos-base-zypper` package is installed. This pattern may be missing on old installations that have only been updated regularly.

The zypper plugin [zypp-boot-plugin](https://manpages.opensuse.org/zypp-boot-plugin) analyses the list of packages which get installed or updated and defines, if a hard reboot is required or a soft-reboot is good enough. `/usr/etc/zypp/zypp-boot-plugin.conf` contains the default configuration, which can be overwritten with drop-ins in `/etc/zypp/zypp-boot-plugin.conf.d/*.conf`

The configuration file `/usr/etc/tukit.conf` defines the system default. The variable `REBOOT_ALLOW_SOFT_REBOOT` is either set to `false`, this means soft-reboot is by default disabled, or `true`, which means a hard reboot is always done if a reboot is requested.

### How to enable

To enable soft-reboot support in transactional-update, create the directory `/etc/tukit.conf.d/` and create a file:

`echo "REBOOT_ALLOW_SOFT_REBOOT=true" > /etc/tukit.conf.d/soft-reboot.conf`.

### How to disable

To disable soft-reboot support in transactional-update, create the directory `/etc/tukit.conf.d/` and create a file:

`echo "REBOOT_ALLOW_SOFT_REBOOT=false" > /etc/tukit.conf.d/soft-reboot.conf`.
