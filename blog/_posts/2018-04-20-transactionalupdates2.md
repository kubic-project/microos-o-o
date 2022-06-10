---
layout: post
title:  "Transactional Updates: Refinements & FAQ"
date:   2018-04-20 17:04:00 +0200
author: Richard Brown
---
After our post a few weeks ago [announcing the availability of transactional updates in openSUSE Tumbleweed & Leap 15]({{ site.baseurl }}{% post_url blog/2018-04-04-transactionalupdates %}) we received a bunch of questions and feedback about this exciting feature.

And of course we didn't rest on our laurels, and we've reacted by implementing as much as we can, as fast as we can. This post covers what we've done, and answers some of the common questions we got about this new feature in openSUSE.

## Renaming the system role

When announcing the new system role that introduces Transactional Updates to the openSUSE distributions it was named **Server with Transactional Updates and Read-Only Root Filesystem**.

![Screenshot](/assets/images/TransactionalRole.png)  
*..it was quite a mouthful*

The name was so long because the intent was to make it obvious that this way of patching a Server is different from the traditional `Server (Text Mode)` role.  
In retrospect we have realised that this was all far too long winded and have now renamed both roles in both openSUSE distributions for clarity.

If you want to install a server that has a read-write root filesystem and is patched using the traditional `zypper` then you can now choose the installation option named **Server**.

For a Server with a read-only root filesystem and patched with `transactional-update` you should now choose the much simpler named **Transactional Server**

![Screenshot](/assets/images/ShorterRoleName.png)  
*Simpler System Role Naming*

## Tuned up Package Selection  

Both the **Server** and **Transactional Server** roles were previously using the `Console` pattern from openSUSE as their default package selection. This pattern provides a number of useful console tools (like `emacs`) but is more than is needed for everyone who wants to use openSUSE as a server. So that has been corrected and both roles are based on the slimmer `Enhanced Base` pattern, which we're also actively working on trimming down to be as efficient as possible. We're aiming to ensure our installations have everything everyone needs to get started with an openSUSE server, but no more than is needed. 
 
## Automatic Transactional Updates by Default

In the **Transactional Server** role, we didn't plan to have `transactional-update` automatically running daily by default. Given the need for reboots to activate each update, our first thought was to avoid automatically rebooting peoples servers without their awareness. But the feedback on this topic has been very clear. Therefore on all installations by default both `transactional-update.timer` and `rebootmgr.service` are now **enabled by default**.

The Transactional Server role now has **fully automated installation of updates** and will reboot between **0330 and 0500** in the morning after an update is installed by default.

![Screenshot](/assets/images/FullyAutomated.png)  
*Transactional Updates now automated by default*

If you wish to modify the timing and behaviour of the reboots, please edit `/etc/rebootmgr.conf` accordingly. Or if you want to be wholly responsible for updating and rebooting your Transactional Servers yourself, feel free to disable the `transactional-server.timer` and `rebootmgr.service` systemd units as you see fit.

## Frequently Asked Questions

**Q: Does rebooting take significantly longer when there is an update?**  
A: Not at all. Thanks to our use of `btrfs` and `snapper`, a system updated by `transactional-update` boots in the same time as a 'normal' openSUSE installation. Rollbacks also take only a fraction of a second to prepare and a reboot to take effect.

**Q: Does transactional-update use the same repositories as zypper? Can I install any package I'd like?**  
A: Yes. Any packages built for your distribution of choice in both official and unofficial repositories should be installable via `transactional-update`. Repository management can be accomplished using `zypper` like regular openSUSE installations. There is the potential for a some packages to have issues installing on a read-only root filesystem. This would suggest a lack of compliance with openSUSE's established Packaging Guidelines, and if anyone discovers such package problems they should [Report a Bug](https://bugs.opensuse.org).

**Q: Any current or future way of using this on a desktop system?**  
A: At the moment we're focusing our efforts on making sure this feature works really well for server use cases. But in practice nothing will prevent any user running `transactional-update pkg in $desktop-pattern-of-their-choice` and installing whatever desktop environment & software they would like. Some people have already done so, and we're sure openSUSE's desktop teams would consider contributions to make a Transactional openSUSE Desktop a great user experience out of the box.

**Q: Can I use transactional-update without a read-only root filesystem?**  
A: Yes, `transactional-update` will work without a read-only root filesystem. However users who choose this approach should be aware that when they reboot their rootfs will be the one created at the time of the `transactional-update`. This potentially means losing any custom changes made in the time between the `transactional-update` and the `reboot`. Therefore the recommendation would be to immediately `reboot` after every update if you're not using a read-only filesystem.

**Q: When can SUSE customers expect to see this make an appearance in a version of SUSE Linux Enterprise?**  
A: While this is a good question, as an openSUSE project we can't provide much of a reliable answer to that. `transactional-update` is already available in [SUSE's CaaS Platform](https://www.suse.com/products/caas-platform/) and we understand there may be some consideration for including this feature in SUSE Linux Enterprise 15 Service Pack 1. SUSE customers interested in this should contact their usual SUSE contact.

**Q: How can I contribute?**  
A: Any changes are welcome to be suggested at the [transactional-update](https://github.com/openSUSE/transactional-update) or [rebootmgr](https://github.com/SUSE/rebootmgr) GitHub projects. If you have any other ideas or questions about Transactional Updates or the Kubic Project in general feel free to get in touch with us by joining our IRC Channel, **#kubic on irc.freenode.org** or by mailing the [openSUSE Factory Mailing list](mailto:opensuse-factory@opensuse.org).

Thanks and *Have a Lot of Fun!*
