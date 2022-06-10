---
layout: post
title:  "MicroOS & Kubic: New Lighter Minimum Hardware Requirements"
date:   2020-11-23 12:15:10 +0100
author: Richard Brown
---

## You Spoke, We Heard

openSUSE MicroOS has been getting a [significant](https://hackaday.com/2020/11/14/microos-is-immutable-linux) [amount](https://news.ycombinator.com/item?id=25094753) of [great](https://www.youtube.com/watch?v=Ve6ygUYobCw&feature=emb_title) attention lately.   
We'd like to thank everyone who has reviewed and commented on what we are doing lately. One bit of clear feedback we received *loud and clear* was that the Minimum Hardware requirement of 20 GB disk space was surprisingly large for an Operating System calling itself **MicroOS**. We agree! And so we've reviewed and retuned that requirement.

## New Minimum Storage Requirements

The New Minimum Supported Storage Requirements for MicroOS are

 * `5 GB` for the read-only `/ (root)` partition, with 20GB as the recommended maximum size.
 * `5 GB` for the read-write `/var` partition, with 40GB as the recommended size, or however large you require for your workloads.

Please Note, a standard installation of the minimal `MicroOS system role` currently uses no more than

 * `450 MB` with bare metal hardware support.
 * `285 MB` without bare metal hardware support.

Therefore these new lighter requirements still ensure that your MicroOS installations have plenty of room for many automated snapshots from `transactional-updates`. These changes will not compromise the promise that MicroOS can be updated and rolled back atomically without worry.

## MicroOS Desktop Differences

The [MicroOS Desktop](https://www.youtube.com/watch?v=cZLckDUDYjw), which is currently in Alpha and being actively developed, has a subtly different minimum requirement, as a result of its different use case.

 * `5 GB` for the read-only `/ (root)` partition, with at least 40GB recommended, or however large you require for your desktop.
 * `/var` and `/home` are provided as read-write `noCoW` sub-volumes as part of the `/ (root)` partition for the storage of containers, flatpaks and user-data.

## Available Now

These changes have all been submitted to openSUSE:Factory, tested in openQA, and will soon be released as part of Snapshot version 20201121. They will soon be available for both [MicroOS](https://en.opensuse.org/Portal:MicroOS/Downloads) and [Kubic](https://en.opensuse.org/Portal:Kubic/Downloads) across all ISO, Cloud, and VM Images.

Thanks and have a lot of fun!

**The MicroOS & Kubic Team**
