---
layout: post
title:  "Introduction to Kubic"
date:   2018-03-22 13:33:00 +0000
author: The Kubic Team
---
Welcome to the webpage and inaugural blog post of the Kubic Project. This post should serve as a basic introduction to Kubic for anyone interested in what we're doing.

<img src="{{ "/assets/images/logo.svg" | prepend: site.baseurl }}" style="width:200px;" class="">

# What is Kubic?

The Kubic Project is a sub-project of the broader [openSUSE Project](https://www.opensuse.org).  
We're focused on new and emerging technologies surrounding containers. We're exploring, developing, adapting and integrating these technologies, helping bring them to the world of openSUSE and helping to improve them directly in their respective upstream projects.

Many of these technologies also serve as upstreams for [SUSE's CaaS Platform](https://www.suse.com/products/caas-platform/) Product.

# Why?

To put it simply, because these technologies are *fun*.

But to try and be a little serious, the ongoing trends with _Containers_, _Micro-Services_, and alternative methods of application delivery are disruptive and changing peoples' expectations. Instead of complicated manual setups, a growing number of apps & services are just a simple 'pull' away, and this changes what users need and expect from their operating systems & surrounding tooling.

The Kubic Project aims to be at the forefront of these trends, taking the best of these new concepts and bringing them to openSUSE while also helping adjust openSUSE to best support these new technologies.

# What are we working on?

As of March 2018 we're currently working on:

 * Transactional Updates
 * MicroOS
 * Tumbleweed Kubic
 * Velum
 * Alternative Container Runtimes (CRI-O, Podman)
 * Rootless Containers

As the world of containers moves very quickly, this list is bound to be incomplete and incorrect for readers in the future, but below is a brief summary of each to give a flavour of what we're working on.

## Transactional Updates

[transactional-update](https://github.com/openSUSE/transactional-update) is a command-line tool that brings **atomic updates** to openSUSE & SUSE distributions.

It leverages our long experience with `btrfs`, `zypper` and `snapper` to update a system _without touching the running system_.

All package updates are prepared as a single operation in a btrfs snapshot. This snapshot is not used until the next reboot.  
Any problems can be _immediately rolled back_ by discarding this transactional snapshot and rebooting again, instantly returning the system to its working order.

When coupled with a **read-only root filesystem**, users are left with a robust running operating system that they can be confident will not change in any way at all while it's running, and can be confidently returned to working order if updates have unintended side-effects.

Transactional Updates with read-only root filesystem are currently available by default in Tumbleweed Kubic and will soon be available as an installation option in both [openSUSE Tumbleweed](https://en.opensuse.org/Portal:Tumbleweed) and [openSUSE Leap 15](https://en.opensuse.org/Portal:Leap).

## MicroOS

[MicroOS](https://en.opensuse.org/Kubic:MicroOS) is the base system for Tumbleweed Kubic.  
It is an [openSUSE Tumbleweed](https://en.opensuse.org/Portal:Tumbleweed) derivative designed to run **containers** and optimised for **large deployments**.

It includes both a read-only root filesystem and _fully automated_ transactional updates out of the box. Its development and release is fully aligned and tested as part of Tumbleweed, meaning any new Tumbleweed release automatically includes updates to Kubic's MicroOS.

MicroOS can currently be installed as by selecting the _System Role_ when installing Tumbleweed Kubic.  
In the future we also intend to offer VM images.

## Tumbleweed Kubic

[Tumbleweed Kubic](http://download.opensuse.org/tumbleweed/iso/openSUSE-Tumbleweed-Kubic-DVD-x86_64-Current.iso) is our **Container-as-a-Service Platform** using **Kubernetes** atop MicroOS.

In addition to the _MicroOS System Role_, Tumbleweed Kubic currently offers the _Unconfigured Cluster Node_ role, allowing users to get started with setting up their own [Kubernetes Cluster](https://kubernetes.io/docs/getting-started-guides/scratch/#bootstrapping-the-cluster).

In the future Tumbleweed Kubic will also offer a further streamlined and automated cluster configuration workflow based on _Velum_.

## Velum

[Velum](https://github.com/kubic-project/velum) is our **Cluster Dashboard & Bootstrap Tool** which will allow you to:

 * Bootstrap a Kubernetes Cluster in a simple WebUI
 * Manage your cluster, including adding & removing nodes, monitor faulty nodes, etc.
 * Setup an update policy to help define when and how you want _Transactional Update_ to run across your cluster.

Velum is under active development and we are hopeful to offer Tumbleweed Kubic images containing Velum in the near future.

## Alternative Container Runtimes

We are currently investigating alternative container runtimes such as [CRI-O](http://cri-o.io/) and its companion tooling [Podman](https://github.com/projectatomic/libpod) as **more lightweight option** for running containers both within Kubernetes and as a stand-alone runtime.

Both are already available in both Tumbleweed & Tumbleweed Kubic today.

## Rootless Containers

This is a project that was spear-headed by our team (based on the work of the
larger container community). The idea was to allow completely unprivileged
users to create containers on their own machines using a standardised container
runtime ([runc][runc]). We also wrote [umoci][umoci] which allows unprivileged
(and privileged) users to operate easily on [OCI][oci] images.

Currently the main interest being worked on (along with some of the containers
community) is the ability to have unprivileged networking using TAP. This would
(theoretically) push us closer to having the possibility of a rootless
Kubernetes deployment. You can keep a close eye on
[rootlesscontaine.rs][rootlesscontainers] if you're interested in more about
this effort.

Rootless containers already work flawlessly on all modern openSUSE
distributions.


[runc]: https://github.com/opencontainers/runc
[umoci]: https://github.com/openSUSE/umoci
[oci]: https://www.opencontainers.org
[rootlesscontainers]: https://rootlesscontaine.rs

# How can I get involved?

Most importantly, like every openSUSE Project, Kubic is an open community.

**We would like your help**.

Our sources can be found on [GitHub](https://github.com/kubic-project).

If you're interested in helping us on anything mentioned here, or have ideas on what we should be looking at, then please get in touch either on our [Mailing List](https://lists.opensuse.org/opensuse-factory/) or on IRC where you can find us in **#Kubic on Freenode**.
