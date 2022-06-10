---
layout: post
title:  "Using Rancher and RKE with MicroOS and Kubic"
date:   2021-02-08 13:20:00 +0100
author: Thorsten Kukuk
---

## Intro

Since [SUSE acquired Rancher Labs](https://www.suse.com/en-en/news/suse-completes-rancher-acquisition/),
it's time to explain how to run Rancher on MicroOS and how to import a Kubic cluster.

I used Rancher 2.5.5 for this, newer versions my have different requirements.

### Rancher with MicroOS

The good news is: Rancher works out of the box on MicroOS.

The necessary steps are:

1. Install MicroOS as base OS (no Container Host system role is necessary)
2. Install docker: `transactional-update pkg install docker`
3. Reboot: `systemctl reboot`
4. Enable and start docker: `systemctl enable --now docker`

From here you can follow the standard [Rancher documentation](https://rancher.com/docs/rancher/v2.x/en/quick-start-guide/deployment/quickstart-manual-setup/)
and install Rancher: `docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher`

### Rancher with RKE and MicroOS

Rancher offers the possibility to setup a new kubernetes cluster using
RKE on an existing, running OS. This section explains how to do that using
MicroOS as the host OS.

While in general, RKE works fine on MicroOS, there could be two pitfalls:
1. MicroOS is using a read-only root filesystem while RKE tries to write to /usr/libexec/kubernetes
2. Rancher reports an error that the API is not reacheable.

The second problem is most likely a docker problem. I suggest to start with
openSUSE MicroOS Build 20210205 or newer, I have never seen this problem with
docker 20.10.3ce introduced with this build. In my case, the reason for the
error message was that IP forwarding didn't got fully enabled by
docker. Please make sure that IP forwarding is enabled for all devices:

```
# sysctl -a |grep \\.forward
net.ipv4.conf.all.forwarding = 1
net.ipv4.conf.default.forwarding = 1
net.ipv4.conf.docker0.forwarding = 1
net.ipv4.conf.eth0.forwarding = 1
net.ipv4.conf.lo.forwarding = 1
```

There are the steps to setup MicroOS for this:

1. Install MicroOS as base OS (no Container Host system role is necessary)
2. Install docker: `transactional-update pkg install docker`
3. Reboot: `systemctl reboot`
4. Enable and start docker: `systemctl enable --now docker`

On the Rancher GUI select "Existing Nodes" of "Create a new Kubernetes cluster
With RKE and existing bare-metal servers or virtual machines"  and follow the
documentation for [Flatcar Container Linux](https://rancher.com/docs/rke/latest/en/os/#flatcar-container-linux).

### Rancher with Kubic

Importing an openSUSE Kubic cluster can be simple or difficult, depending
on which kubernetes version your cluster is running.
openSUSE Kubic currently comes with kuberenetes 1.20.2 as default. Rancher
only works with kuberenetes up to 1.19.7, it does not work with 1.20.2 as of
today. So if you haven't updated your cluster yet to 1.20.x, you can go to
`Register an existing Kubernetes cluster`, select `Other Cluster` and follow
the workflow.

At the end, Rancher will come up with two errors:
1. Alert: Component controller-manager is unhealthy.
2. Alert: Component scheduler is unhealthy.

You can ignore this errors: Rancher uses a deprecated interface, which kubeadm disables by default.
