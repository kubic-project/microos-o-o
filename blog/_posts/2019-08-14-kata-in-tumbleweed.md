---
layout: post
title:  "Kata Containers now available in Tumbleweed"
date:   2019-08-15 11:00:00 +0200
author: Marco Vedovati
---

![Kata](/assets/images/kata-horz-onwhite.png)

Kata Containers is an open source container runtime that is crafted to seamlessly
plug into the containers ecosystem.

We are now excited to announce that the Kata Containers packages are finally available
in the official openSUSE Tumbleweed repository.

It is worthwhile to spend few words explaining why this is a great news, considering
the role of Kata Containers (a.k.a. Kata) in fulfilling the need for security in the
containers ecosystem, and given its importance for openSUSE and Kubic.


## What is Kata
As already mentioned, Kata is a container runtime focusing on security and on ease of
integration with the existing containers ecosystem. If you are wondering what's a
container runtime, [this blog post by Sascha](https://www.suse.com/c/demystifying-containers-part-ii-container-runtimes/)
will give you a clear introduction about the topic.

Kata should be used when running container images whose source is not fully trusted,
or when allowing other users to run their own containers on your platform.

Traditionally, containers share the same physical and operating system (OS) resources
with host processes, and specific kernel features such as namespaces are used to provide
an isolation layer between host and container processes.
By contrast, Kata containers run inside lightweight virtual machines, adding an
extra isolation and security layer, that minimizes the host attack surface and mitigates
the consequences of containers breakout.
Despite this extra layer, Kata achieves impressive runtime performances thanks to
KVM hardware virtualization, and when configured to use a minimalist virtual machine
manager (VMM) like Firecracker, a high density of microVM can be packed on a single host.

If you want to know more about Kata features and performances:
- katacontainers.io is a great starting point.
- For something more SUSE oriented, Flavio [gave a interesting talk about Kata](https://youtu.be/EmOcxtCYzjk) at SUSECON 2019,
- Kata folks hang out on katacontainers.slack.com, and will be happy to answer any quesitons.

## Why is it important for Kubic and openSUSE
SUSE has been an early and relevant open source contributor to containers projects,
believing that this technology is the future way of deploying and running software.

The most relevant example is the [openSUSE Kubic project](https://kubic.opensuse.org/),
that's a certified Kubernetes distribution and a set of container-related technologies
built by the openSUSE community.

We have also been working for some time in well known container projects, like runC,
libpod and CRI-O, and since a year we also collaborate with Kata.

Kata complements other more popular ways to run containers, so it makes sense for
us to work on improving it and to assure it can smoothly plug with our products.

## How to use
While Kata may be used as a standalone piece of software, it's intended use is serve
as a runtime when integrated in a container engine like [Podman](https://podman.io/) or [CRI-O](https://cri-o.io/).

This section shows a quick and easy way to spin up a Kata container using Podman
on openSUSE Tumbleweed.

First, install the Kata packages:
```
$ sudo zypper in katacontainers
```

Make sure your system is providing the needed set of hardware virtualization features
required by Kata:
```
$ sudo kata-runtime kata-check
```
If no errors are reported, great! Your system is now ready to run Kata Containers.

If you haven't already, install podman with:
```
$ sudo zypper in podman
```

That' all. Try running a your first Kata container with:
```
$ sudo podman run -it --rm --runtime=/usr/bin/kata-runtime opensuse/leap uname -a
Linux ab511687b1ed 5.2.5-1-kvmsmall #1 SMP Wed Jul 31 10:41:36 UTC 2019 (79b6a9c) x86_64 x86_64 x86_64 GNU/Linux
```

### Differences with runC
Now that you have Kata up and running, let's see some of the differences between
Kata and runC, the most popular container runtime.

When starting a container with runC, container processes can be seen in the host
processes tree:
```
...
10212 ?        Ssl    0:00 /usr/lib/podman/bin/conmon -s -c <ctr-id> -u <ctr-id>
10236 ?        Ss     0:00  \_ nginx: master process nginx -g daemon off;
10255 ?        S      0:00      \_ nginx: worker process
10256 ?        S      0:00      \_ nginx: worker process
10257 ?        S      0:00      \_ nginx: worker process
10258 ?        S      0:00      \_ nginx: worker process
...
```

With Kata, container processes are instead running in a dedicated VM, so they are
not sharing OS resources with the host:
```
...
10651 ?        Ssl    0:00 /home/marco/go/src/github.com/containers/conmon/bin/conmon -s -c <ctr-id> -u <ctr-id>
10703 ?        Sl     0:01  \_ /usr/bin/qemu-system-x86_64 -name sandbox-<ctr-id> -uuid e54ee910-2927-456e-a180-836b92ce5e7a -machine pc,accel=kvm,kernel_ir
10709 ?        Ssl    0:00  \_ /usr/lib/kata-containers/kata-proxy -listen-socket unix:///run/vc/sbs/<ctr-id>/proxy.sock -mux-socket /run/vc/vm/829d8fe0680b
10729 ?        Sl     0:00  \_ /usr/lib/kata-containers/kata-shim -agent unix:///run/vc/sbs/<ctr-id>/proxy.sock -container <ctr-id>
...
```

## Future plans
We are continuing to work to offer you a great user experience when using Kata on openSUSE by:
- improving packages quality and stability,
- delivering periodic releases,
- making sure that Kata well integrates with the other container projects, like Podman and CRI-O.

As a longer term goal, we will integrate Kata in the Kubic distribution and in CaaSP,
to make them some of the most complete and secure solutions to manage containers.

