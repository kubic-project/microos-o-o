---
layout: post
title:  "Quickstart in Full Disk Encryption with TPM and YaST2"
date:   2024-09-03 14:07:00 +0200
author: Thorsten Kukuk
---

## Intro

This is a quick start guide for Full Disk Encryption with TPM or FIDO2 and
YaST2 on openSUSE MicroOS. It focuses on the few steps to
install openSUSE MicroOS with YaST2 and using Full Disk Encryption
secured by a TPM2 chip and [measured boot](https://en.opensuse.org/Portal:MicroOS/RemoteAttestation#Measured_boot)
or a FIDO2 key.

## Hardware Requirement:
- UEFI Firmware
- TPM2 Chip or FIDO2 key which supports the hmac-secret extension
- 2GB Memory

## Installation of openSUSE MicroOS

### Boot installation media

To avoid the use of the YaST2 expert partitioner, the default LUKS key
derivation function from YaST2 will be used and three unused subvolumes
(`/boot/grub2/i386-pc`, `/boot/grub2/x86_64-efi`, `/boot/writeable`) will
be created.
If you are familiar with the YaST2 expert partitioner, you can remove the
three btrfs subvolumes and change the LUKS key derivation function to
`argon2id` already during installation.

* Follow the workflow until "Installation Settings"
* Installation Settings:
  * Partitioning:
    * Select "Guided Setup" and "Enable Disk Encryption", keep the other defaults
  * Booting:
    * Change Boot Loader Type from "GRUB2 for EFI" to "Systemd Boot", ignore "Systemd-boot support is work in progress" and continue
* Finish Installation

The order of adjusting the partitioning and changing the bootloader is important.

### Finish FDE Setup

Boot new system
* Enter passphrase to unlock disk during boot
* Login
* Enroll system:
  * With TPM2 chip: `sdbootutil enroll --method tpm2`
  * With FIDO2 key: `sdbootutil enroll --method fido2`
* Optional, but recommended:
  * Upgrade your LUKS key derivation function (do that for every encrypted device listed in `/etc/crypttab`):
  ```
          # cryptsetup luksConvertKey /dev/vdaX --pbkdf argon2id
          # cryptsetup luksConvertKey /dev/vdaY --pbkdf argon2id
  ```

## Adjusting kernel boot parameters

The configuration file for kernel command line options is `/etc/kernel/cmdline`.

After editing this file, call `sdbootutil update-all-entries` to update the
bootloader configuration. If that option does not exist yet or does not work,
a workaround is: `sdbootutil remove-all-kernels && sdbootutil add-all-kernels`.

## Re-enrollment
If the prediction system fails, a new policy must be created for the new measurements to replace the policy stored in the TPM2.

If you have a recovery PIN:
```
  # sdbootutil --ask-pin update-predictions
```

If you don't have the recovery PIN, you can set one with this steps:
```
  # sdbootutil unenroll --method=tpm2
  # PIN=<new recovery PIN> sdbootutil enroll --method=tpm2
```

## Virtual Machines

If your machine is a VM, it is recommended to remove the "0" from the `FDE_SEAL_PCR_LIST` variable in `/etc/sysconfig/fde-tools`. An update of the hypervisor can change PCR0. Since such an update is not visible inside the VM, the PCR values cannot be updated. As result, the disk cannot be decrypted automatically at the next boot, the recovery key needs to be entered and a manual re-enrollment is necessary.

## Next Steps

The next steps will be:

* Support grub2-BLS (grub2 following the [Boot Loader Specification](https://uapi-group.org/specifications/specs/boot_loader_specification/))
* Add support to the installers (YaST2 and Agama)
* Make this the default if a TPM2 chip is present

Any help is welcome!

## Further Documentation

* [Full Disk Encryption (FDE)](https://en.opensuse.org/Portal:MicroOS/FDE)
* [Systemd-fde](https://en.opensuse.org/Systemd-fde)
* [Systemd-boot and Full Disk Encryption with TPM and FIDO2](https://microos.opensuse.org/blog/2023-12-20-sdboot-fde/)
