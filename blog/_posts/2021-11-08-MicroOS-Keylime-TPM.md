---
layout: post
title:  "MicroOS Remote Attestation with TPM and Keylime"
date:   2021-11-08 13:20:00 +0100
author: Alberto Planas
---

## Introduction

During 2021 we have been starting to focus more in security for
[MicroOS](https://microos.opensuse.org/).  By default MicroOS is a
fairly secure distribution: during the development all the changes are
publicly reviewed, fixes (including CVEs) are integrated first (or at
the same time) in Tumbleweed, we have read-only root system and a tool
to recover old snapshots, and periodically the security team audit
some of the new components.  Also, the move from AppArmor to SELinux
should help to standardize the security management.

But we really want to rise the bar when it is possible.  For example,
we are starting to think on how to enable
[IMA/EVM](https://en.opensuse.org/SDB:Ima_evm) properly in the
distribution, or what alternatives we have for [full disk encryption
supported by a
TPM](https://0pointer.net/blog/unlocking-luks2-volumes-with-tpm2-fido2-pkcs11-security-hardware-on-systemd-248.html).
There are some evaluation on
[dm-verity](https://www.kernel.org/doc/html/latest/admin-guide/device-mapper/verity.html)
inside the new [Transactional Image
Update](https://github.com/thkukuk/tiu) installer.

Another area where we make progress in MicroOS is how to measure the
health of our systems, detect remotely when an unauthorized change has
been made (remote attestation), and actuate over it globally and as
fast as possible.


### TPM as a root of trust

Today all our devices (laptops, desktops, servers, phone or tablets)
includes a cryptoprocessor known as TPM (from the initials Trusted
Platform Module).  Sometimes is inside the CPU, but can also be found
soldered in the motherboard or implemented as a software in our
firmware.  Those co-processors are really cheap (and sadly slow), but
very useful when we want to design software that requires a hardware
based root of trust.

For example, imagine that we want to encrypt the disk but we do not
want to be asked for the password at the beginning of the boot.  We
should need some trusted component that can provide the password very
early in the boot process, and that the system can - somehow -
validate that it is the real deal and not some other agent trying to
impersonate it.  A TPM provides mechanisms to do this validation, and
if every this goes OK to unseal the password to the kernel so it can
decrypt the disk.

This same role is required for many other operations, like accessing
to a VPN that requires the validation of the combination machine /
user.  Because of how the TPMs are designed, they can generate keys
that we can check that comes from this specific TPM and no other.
This property is valuable to open the access to the internal network,
for example.

Another activity where a root of trust is require is when we need to
validate the health of our systems via _measured boot_.

### Measured Boot and remote attestation

The general mechanism goes like this: during the boot process, a very
early piece of the firmware takes care of initializing some hardware
components and setting some clocks, and when is done, before
delegating the execution to the next stage (maybe some early stage in
the UEFI firmware), it will load in memory this next stage and will
calculate a hash of it (like SHA256), and will communicate this
information to some piece of hardware that we trust (the TPM in this
case).  This component can now delegate the boot process to this
second stage, that will do the same operation when needs to move to
the third stage (load, measure and communicate the measurement to the
TPM), and this goes on until Grub2 enter in action and load the
kernel.

If we do this we effectively have a measurement of every step in the
bootloader chain, from the most early stage deep in the firmware until
(and included) the kernel.  This process is known "measured boot".
And the TPM have a record of all those measurements!

Actually I am lying (well, sort off, as we will see a bit later).

TPMs are cheap and do not have a lot of space inside, so the TPM by
itself cannot have the full record of measurement.  Inside the TPM we
have some registers known as platform configuration registers (PCR)
that have one feature: we can read them, but we cannot directly write
on them.  To change the value of a register we have an operation known
as _extension_, that can be viewed as taking the current value of the
register, attaching at the end the value of the hash that we want to
extend with (in this case the measurement done), and calculating the
hash of the full string (again, like SHA256).  This new value
calculated is the one that will be stored in the PCR instead.

This _extend_ operation makes that the current value of the PCR
depends on all the previous values of it and the values (measurements)
used before.  So in order to replicate a value we need to know the
correct measurement of each stage of the boot chain (including the
initial value of the PCR after the reset, that is usually 0x00..0).

The goal here is that once all is booted, the user can ask for those
PCR values to the TPM via another operation named _quote_.  This
operation returns a report signed by a key that only the TPM knows,
and that I can validate to see if, indeed, comes from my TPM or not.
I can later compare the PCR values with the ones that I expect for my
current version of UEFI, Grub2 and kernel.  If they match I know for
sure that the boot chain has not been tampered, and if do not match
... well ... we have been hacked (or we have an unauthorized updated
somewhere that we need to check).

To be honest, comparing raw PCR values is hard.  If we do not know all
the measurements of our boot chain, they are impossible to predict.

This is why for each measurement, from the UEFI until the kernel,
besides extending a PCR register in the TPM we are also storing in
memory a log of all those measurements.  We register (in the normal
memory) some bits extra of information used during the measurement,
like the PCR number and the value used for the extension.  We can also
see in the log some signatures found (and expected if we are using
secure boot), text data (for example the kernel command line used in
Grub2), etc.

This log will be passed between stages until it reaches the kernel,
that will make is available to the user space via the security file
system.  This log is known as the _event log_.

Another point here is that we can now ask, remotely from a different
machine, the current values of the PCRs via a quote to the TPM, and
the full content of the event log.  With this information we can
attest remotely that the system has not been compromised during the
boot process. Of course, this is known as _remote attestation_.


### Keylime

If we have a device with a TPM we can go to the BIOS / UEFI boot menu
and activate it.  After that, every boot of this system will be
measured as described here, and we can do the attestation ourselves
requesting a _quote_ to TPM (via the `tpm2.0-tool` package in MicroOS)
and comparing the values with the _event log_.

But this is cannot scale properly, we need a tool to help us doing
this automatically.

[Keylime](https://keylime.dev/) is an open source project designed to
do exactly what we need: do remote attestation of our devices, using a
TPM as a root of trust of all the measurements.

In Keylime we have some services available.  The _agent_ is the
service that needs to be installed in all of the nodes that we want to
monitor.  This service is responsible of collecting some data (TPM
quotes, event log, IMA logs, etc) under demand.

This information is requested by another service known as _verifier_,
that will validate this data based on some user provide information.

For example, it will check that the PCR values are the expected ones.
It can also inspect the event log using an user provided policy (as a
Python code) that will search for some signatures, will see the Grub
menu used for booting and will compare some of the measurements that
known of (like the kernel one, for example).

Also, if IMA is enabled in the remote node, it can validate the hashes
of all the programs and services that we are running!

With Keylime we can deploy secrets to our nodes (like certificates or
keys), and we can execute user-defined actions when we detect an
unauthorized change in our nodes.

For example, if Keylime detect the execution of a program with a
different IMA hash that the one expected, we can execute a program
that will try to isolate this node from their peers, and will revoke
any access to the shared resources in the network, like databases.

The good news is that Keylime has been integrated in MicroOS via two
new system roles in YaST.  During the installation we can see know two
new roles, one should be used for the nodes that we want to monitor
(_agent_ role), and the other for the node that will take care of
collecting the information of the agents, doing the verification and
triggering the revocation actions when required (_verifier_ role).

Those roles make very easy to start doing remote attestation in
MicroOS today and the process is fully documented.


### More information!

We documented all this information (and more) in the [MicroOS
portal](https://en.opensuse.org/Portal:MicroOS/RemoteAttestation).
There you can find a more technical description, including how to
configure the _agent_ services in order that they can find the
_verifier_, how to enable IMA in the nodes and prepare a white list of
hashes, and how to write programs that can acts when an intrusion has
been detected.

There is also some more details about the TPM and how to check if they
are present in the system and recognized by MicroOS.

Also recently we had the SUSE Labs 2021 conference, and all the videos
has been
[published](https://www.youtube.com/playlist?list=PL4ibkKyj5eYTDzuV0aYfUrFuXPC9GAmrs)
recently, including a talk about [Keylime and TPM in the context of
MicroOS](https://www.youtube.com/watch?v=6F2mxG4YRKg&list=PL4ibkKyj5eYTDzuV0aYfUrFuXPC9GAmrs&index=18)
that you can check.  In the proceedings of this conference there is
also a
[paper](https://github.com/aplanas/keylime-suselabs/blob/main/keylime.pdf)
that can be useful to complement this topic!
