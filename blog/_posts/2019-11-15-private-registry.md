---
layout: post
title:  "Private and Air-Gap registry for openSUSE Kubic"
date:   2019-11-15 12:47:00 +0100
author: Thorsten Kukuk
---

## Intro

Sometimes there are occasions where direct internet access is not possible
(proxy/offline/airgapped). Even in this setups it is possible to deploy
and use Kubernetes with [openSUSE Kubic](https://kubic.opensuse.org) and a
local private registry.

In this blog I will explain how to setup a local server which acts as
private registry providing all the container images needed to deploy
Kubernetes with openSUSE Kubic.

## Installation

### OS

As OS for our host serving the required services
[openSUSE MicroOS](https://en.opensuse.org/Kubic:MicroOS) is used, which needs
to be installed with the `MicroOS Container Host` system role.

The following additional packages are required and need to be installed:
  * container-registry-systemd
  * mirror-registry
  * skopeo with the latest sync patches as provided in openSUSE Tumbleweed
  * reg (optional, for testing and debugging)

```
# transactional-update pkg install mirror-registry skopeo container-registry-systemd reg
# systemctl reboot
```
After the reboot the tools are available and we can start with the setup
of the registry.

### Registry Setup

There is a script to setup a local private registry running in containers.
By default, everybody can list and pull images from it, but only the admin
is allowed to push new images.

```
# setup-container-registry
```

There is no default password for the admin account, which means
it is necessary to set one to be able to sync the containers to
this registries. The password must be a bcrypt (blowfish) hash:

```
# htpasswd -nB admin
```

Modify the password entry for the admin account and set the new hash:

```
# nano /etc/registry/auth_config.yml
```

Additional, the ACL rules can be adjusted.

Start the containers for the registry:

```
# systemctl start container-registry
# systemctl start registry-auth_server
```

Now your registry is running and should work. This can be verified with:

```
# reg ls localhost
Repositories for localhost
REPO                TAGS
```

### Certificates

By default `setup-container-registry` will create self signed
certificates, which are valid for 1,5 year and stored in
`/etc/registry/certs`. They can be replaced with official certificates.

If the self signed certificates are used, the public CA certificate needs
to be installed on all machines, which should access the registry. Copy
`/etc/registry/certs/ContainerRegistryCA.crt` to `/etc/pki/trust/anchors/`
and run `update-ca-certificates` on every machine.

### List of container images

The tool `mirror-registry` will be used to create a list of containers
which needs to be mirrored. It will analyse a remote registry and
create a yaml file with all containers and tags matching a regex to
sync with `skopeo` to a private registry.

This tool can run on any architecture, the target platform can be
specified as argument. In contrast to this, skopeo needs to run on
the target architecture. Else it will fail or sync the wrong container
image if there are multi-architecture container images in the source
repository.

All container images below `registry.opensuse.org/kubic/` are required.
Additional, it could be from help to mirror all official openSUSE
container images from `registry.opensuse.org/opensuse/`, too.
Else utilities like `toolbox` will not work.

The commnd to create a list of container images is:

```
# mirror-registry --out kubic-full.yaml registry.opensuse.org "(^kubic|^opensuse/([^/]+)$)"
```

This command will mirror all builds of all versions of the required
container images. But most of the time it is enough to mirror the latest
build of every version:

```
# mirror-registry --out kubic-small.yaml --minimize registry.opensuse.org "(^kubic|^opensuse/([^/]+)$)"
```

`kubic-small.yaml` can now be further tuned. If e.g. weave is used for the POD
network, flannel and the cilium containers don't need to be mirrored
and can be deleted from the list.

#### Private registry

From a host, which has access to the source registry and the internal
private registry, the container images can now be synced with skopeo:

```
# skopeo sync --scoped --src yaml --dest docker --dest-creds admin:password kubic-small.yaml registry.local
```

The `--scoped` option will prefix the internal registry name to the
current external repository name, so the repository will be
`registry.opensuse.org/...` and the pull command
`podman pull registry.local/registry.opensuse.org/...`
to avoid clashes with the namespace of the internal registry or by
mirroring additional registries.

The result can be verified with:

```
# reg ls registry.local
```

skopeo needs recent enough patches adding the `sync` option. This patches
are already part of openSUSE Tumbleweed, openSUSE MicroOS and openSUSE Kubic,
but not of SUSE Linux Enterprise Server 15.

#### Airgapped/offline registry

If there is no host which can access both registries, the external and the
internal one, the container images needs to be copied to a disk, which then
needs to be transfered into the internal network.

Copy the container images to an external disk:
```
# skopeo sync --scoped --src yaml --dest dir kubic-small.yaml /media/external-disk
```

Take the external disk to the internal registry machine and import
the container images:
```
# skopeo sync --scoped --src dir --dest docker --dest-creds admin:password /media/external-disk localhost
```

Verify the result:
```
# reg ls registry.local
```

## Node configuration

After the private registry is running and contains all container images,
cri-o and podman need to be aware of this registry and pull the images
from it.

Both tools share the same configuration file for this: `/etc/containers/registries.conf`.

The default configuration file is using the v1 syntax, but v2 is needed.
The documentation can be found
[here](https://github.com/containers/image/blob/master/docs/containers-registries.conf.5.md).
A working configuration template, which needs to be deployed on every
kubernetes node, could look like:

```
[[registry]]
prefix = "registry.opensuse.org/kubic"
location = "registry.local/registry.opensuse.org/kubic"
insecure = false
blocked = false

[[registry]]
prefix = "registry.opensuse.org/opensuse"
location = "registry.local/registry.opensuse.org/opensuse"
insecure = false
blocked = false
```
  * insecure: `true` or `false`. By default TLS is required when retrieving images from a registry. If insecure is set to true, unencrypted HTTP as well as TLS connections with untrusted certificates are allowed.
  * blocked : `true` or `false`. If true, pulling images with matching names is forbidden.

After updating the configuration file, cri-o needs to be restarted on that
node if it is running:
```
# systemctl status crio
* crio.service - Open Container Initiative Daemon
   Loaded: loaded (/usr/lib/systemd/system/crio.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-11-02 18:09:23 UTC; 8min ago
[...]
# systemctl stop crio
# systemctl start crio
```

Now you can deploy kubernetes!

## Configuration files

Following configuration files define the behavior of the registry and the
authentication server:

  * /etc/registry/config.yml - configuration file for registry
  * /etc/registry/auth_config.yml - configuration file for auth server
  * /etc/sysconfig/container-registry - configuration file for the two containers
