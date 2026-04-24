---
layout: post
title:  "Managing System Extensions with sysextmgrcli"
date:   2026-04-23 17:27:00 +0200
author: Stefan Schubert
---

# Managing System Extensions on openSUSE MicroOS with `sysextmgrcli`

If you are running **openSUSE MicroOS**, you already know the drill: the root filesystem is read-only,
and transactional updates are the law of the land.
But what happens when you need to add software or system extensions without rebooting or messing with
your base OS layers?

Enter **System Extensions (sysext images)** and the utility designed to make them manageable: `sysextmgrcli`.

---

## What is `sysextmgrcli`?

At its core, `sysextmgrcli` is a command-line client for managing `systemd-sysext` images and has been written
by Thorsten Kukuk. It is designed specifically to play nice with the atomic nature of MicroOS.

Instead of forcing you to use `sudo` for every query, it talks to a background daemon (`sysextmgrd`) via
**Varlink**. This architecture allows unprivileged users to list existing system extension images without
needing root permissions, while the daemon handles the heavy lifting of downloads and verification via `systemd-pull`.

## The Architecture: Smart Snapshots

One of the cleverest things about `sysextmgrcli` is how it handles storage to be efficient and "rollback-safe":

* **/var/lib/sysext-store**: This is where the actual image files live. Since `/var` is a separate subvolume
shared across all Btrfs snapshots, you only store the image once, saving disk space.
* **/etc/extensions**: This directory contains **symlinks** to the images in the store. Because `/etc` is part of
your root snapshot, the extensions are tied to your current system state. 

**Why does this matter?** If you perform a system rollback, your symlinks roll back too. This ensures the active
sysext images always match the OS version you are currently booted into.

---

## Essential Commands

Getting started is straightforward. Here are the primary commands you'll use to manage your extensions:

### 1. Listing and Checking Images
Want to see what’s available or if your images are compatible with your current OS version?
```
# List all images and report compatibility
sysextmgrcli list

# Check for updates and verify compatibility
sysextmgrcli check
```

### 2. Installing New Extensions
You can install by providing a name and a source URL. The tool automatically handles SHA256 verification and
checks if it fits your OS.

```
# --url is optional (default: https://download.opensuse.org/tumbleweed/appliances/ )
sysextmgrcli install [NAME] --url [https://your-image-repo.com](https://your-image-repo.com)
```

### 3. Maintenance and Updates
Updates are handled by comparing local files against remote manifests. If a newer version matches your current snapshot, it gets pulled down and symlinked.

```
# Update existing images to the latest compatible versions
sysextmgrcli update

# Clean up: Remove images in the store that are no longer referenced by any snapshot
sysextmgrcli cleanup
```

## The "Activation" Catch
It is important to note that sysextmgrcli is a manager, not an activator. It handles the logistics: downloading, version checking, and symlinking. To actually "plug in" the extensions to your running system, you still use standard systemd-sysext commands:

* Manual activation: `systemd-sysext merge`

* Manual deactivation: `systemd-sysext unmerge`

* Enable at boot: `systemctl enable systemd-sysext.service`

## Available default system extention (sysext) images:

* debug (babeltrace, gdb, ltrace, strace, traceroute)
* gcc (cpp, gcc, make, patch, update-alternatives)
* git (git, git-core)

## Summary

### You need `git` on your **openSUSE MicroOS** ?
Just install the `git` system extention with `sysextmgrcli`, activate and use it...

### You do not need 'git' anymore on your system ?
Just deactivate it and it is not available anymore...

sysextmgrcli bridges the gap between static immutable infrastructure and the need for flexible system additions. By leveraging the Btrfs directory structure of MicroOS, it ensures your system remains clean, version-synced, and easy to manage.

