---
layout: post
title:  "New default: tmpfs on /tmp"
date:   2020-07-27 15:39:00 +0200
author: Thorsten Kukuk
---

## Intro

We made an important change for our Container Host OS [openSUSE
MicroOS](https://en.opensuse.org/Portal:MicroOS), which our Kubernetes
platform [openSUSE Kubic](https://kubic.opensuse.org) will inherit since it is
based on openSUSE MicroOS: we use now `tmpfs` for `/tmp`.


`tmpfs` is a temporary filesystem that resides in memory. Mounting directories
as `tmpfs` can be an effective way of speeding up accesses to their files and
to ensure that their contents are automatically cleared upon reboot.

A fresh installation will use `tmpfs` for `/tmp` by default. Old installations
needs to be converted to this manually, but it is still possible to switch
back to use disk space for `/tmp`. This is especially useful and important, if
big files are stored in `/tmp`.

If temporary files or directories are needed below `/tmp`, this can be created
at boot by using `tmpfiles.d`.
But never store important files in `/tmp`, they will not survive the next
reboot.

### Converting old installations to use tmpfs

As `tmpfs` will be mounted on top of `/tmp`, existing files will be no longer
accessible. The following steps will cleanup `/tmp` and enable `/tmpfs`:

1. Backup all important files currently stored in `/tmp`!
2. Remove the line for `/tmp` from /etc/fstab
3. Remove all files in `/tmp`
4. Reboot

After reboot, `tmpfs` should be used for `/tmp`.

### Using disk space for `/tmp`

After creating a new btrfs subvolume for `/tmp` and adding this to
`/etc/fstab`, tmpfs will no longer be used for `/tmp`.

The easierst way to archive this is to use mksubvolume from snapper 0.8.12 or
newer:

```
# transactional-update shell
transactional update # mksubvolume /tmp
transactional update # exit
# systemctl reboot
```

Afterwards, all files are stored again on the disk and will survive a reboot.

## Future plans

In the future we plan to harden the system by default even more, e.g. by
marking `/tmp` and other write-able parts of the filesystem with `noexec`.
