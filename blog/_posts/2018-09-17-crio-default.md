---
layout: post
title:  "CRI-O is now our default container runtime interface"
date:   2018-09-17 15:18:00 +0200
author: Richard Brown
---

![CRI-O](/assets/images/criologo.svg)

We're really excited to announce that as of today, we now officially supports the CRI-O Container Runtime Interface as our default way of interfacing with containers on your Kubic systems!

## Um that's great, but what is a Container Runtime Interface?

Contrary to what you might have heard, there are more ways to run containers than just the `docker` tool. In fact there are an increasing number of options, such as `runc`, `rkt`, `frakti`, `cri-containerd` and more. Most of these follow the [OCI](https://www.opencontainers.org/) standard defining how the runtimes start and run your containers, but they lack a standard way of interfacing with an orchestrator. This makes things complicated for tools like kubernetes, which run on top of a container runtime to provide you with orchestration, high availability, and management.

Kubernetes therefore introduced a standard API to be able to talk to and manage a container runtime. This API is called the [Container Runtime Interface (CRI)](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/).

Existing container runtimes like Docker use a "shim" to interface between Kubernetes and the runtime, but there is another way, using an interface that was designed to work with CRI natively. And that is where CRI-O comes into the picture.

## Introduction to CRI-O

Started little over a year ago, CRI-O began as a Kubernetes incubator project, implementing the CRI Interface for OCI compliant runtimes. Using the lightweight `runc` runtime to actually run the containers, the simplest way of describing CRI-O would be as a lightweight alternative to the Docker engine, especially designed for running with Kubernetes.

As of [6th Sept 2018](https://twitter.com/fatherlinux/status/1037810496643244039) CRI-O is no longer an incubator project, but now an official part of the Kubernetes family of tools.

## Why CRI-O?

There are a lot of reasons the Kubic project love CRI-O, but to give a **Top 4** some of the largest reasons include:

- **A Truly Open Project:** As already mentioned, CRI-O is operated as part of the broader Kubernetes community. There is a [broad collection](https://github.com/kubernetes-sigs/cri-o/graphs/contributors) of contributors from companies including Red Hat, SUSE, Intel, Google, Alibaba, IBM and more. The project is run in a way that all these different stakeholders can actively propose changes and can expect to see them merged, or at least spur steps into that direction. [This is harder to say of other similar projects](https://github.com/moby/moby/pull/34319). 
- **Lightweight:** CRI-O is made of lots of small components, each with specific roles, working together with other pieces to give you a fully functional container experience. In comparison, the Docker Engine is a heavyweight daemon which is communicated to using the `docker` CLI tool in a client/server fashion. You need to have the Daemon running before you can use the CLI, and if that daemon is dead, so is your ability to do anything with your containers.
- **More Securable:** Every container run using the Docker CLI is a 'child' of that large Docker Daemon. This complicates or outright prevents the use of tooling like cgroups & security constraints to provide an extra layer of protection to your containers. As CRI-O containers are children of the process that spawned it (not the daemon) they're fully compatible with these tools without complication. This is not only cool for Kubernetes, but also when using CRI-O with Podman, but more about that later...
- **Aligned with Kubernetes:** As an official Kubernetes project, CRI-O releases in lock step with Kubernetes, with similar version numbers. ie. CRI-O 1.11.x works with Kubernetes 1.11.x. This is hugely beneficial for a project like Kubic where we're rolling and want to keep everything working together at the latest versions. On the other side of the fence, Kubernetes currently only officially supports Docker versions 17.03.x, now well over 1 year old and far behind the 18.06.x version we currently have in Kubic.

## CRI-O and Kubernetes

Given one of the main roles of Kubic is to run Kubernetes, as of today, Kubic's **kubeadm system role** is now designed to use CRI-O by default.

[Our documentation has been updated](https://en.opensuse.org/Kubic:kubeadm) to reflect the new CRI-O way of doing things.

The simplest way of describing it would be that we now have less steps than [before]({{ site.baseurl }}{% post_url blog/2018-08-20-kubeadm-intro %}).  
**You can now initialise your master node with a single command immediately after installation.**  
But you need to remember to add `--cri-socket=/var/run/crio/crio.sock` to your `kubeadm init` and `join` commands. *(We're looking into ways to streamline this)*. 

## CRI-O and MicroOS

Kubic is about more than Kubernetes, and our **MicroOS system role** is a perfect platform for running containers on stand-alone machines. That too now includes CRI-O as it's default runtime interface.

In order to make use of CRI-O without Kubernetes, you need a command-line tool, and that tool is known as `podman`. It is now installed by default on Kubic MicroOS.

## Podman

[Podman]({{ site.baseurl }}{% post_url blog/2018-03-25-podman %}) has been available in Tumbleweed & Kubic for some time. Put simply, it is to CRI-O what the Docker CLI tool is to the Docker Engine daemon. It even has a very similar syntax.

- Use `podman run` to run containers in the same way you'd expect from `docker run`
- `podman pull` pulls containers from registries, just like `docker pull`, and by default our `podman` is configured to use the same Docker Hub as many users would expect.
- Some `podman` commands have additional functionality compared to their `docker` equivalents, such as `podman rm --all` and `podman rmi --all` which will remove all of your containers and their images respectively.
- [A full crib-sheet of podman commands and their docker equivalents is available](https://github.com/containers/libpod/blob/master/transfer.md)

Podman also benefits from CRI-Os more lightweight architecture. Because every Podman container is a direct child of the `podman` command that created it, it's trivial to use `podman` as part of systemd services. This be combined with systemd features like socket activation to do really cool things like starting your container only when users try to access it!
 
## What about Docker?

As excited as we are about CRI-O and Podman, we're not blind to the reality that many users just won't care and will be more comfortable running the well known `docker` tool.

For the basic use case of running containers, both `docker` and `podman` can co-exist on a system safely. Therefore it will still be available and installed by default on Kubic **MicroOS**.  
If you wish to remove it, just run `transactional-update pkg rm -u docker-kubic` and reboot.

The Docker Engine doesn't co-exist with CRI-O quite so well in the Kubernetes scenario, so we do not install both by default on the **kubeadm system role**.  
We still wish to support users wishing to use the Docker Engine with Kubernetes. Therefore to swap from CRI-O to the Docker Engine just run `transactional-update pkg in patterns-caasp-alt-container-runtime -cri-o-kubeadm-criconfig` and reboot.

Alternatively if you're installing Kubic from the installation media you can deselect the "Container Runtime" and instead choose the "Alternative Container Runtime" pattern from the "Software" option as part of the installation.

Regardless of which runtime you choose to use, thanks for using Kubic and please join in, send us your feedback, code, and other contributions, and remember, **have a lot of fun!**.
