---
layout: post
title:  "The Current State of MicroOS Desktop Plasma"
date:   2023-04-01 20:40:00 -0800
author: Shawn W Dunn
---

## Current state of things

MicroOS Desktop Plasma is still in an "Alpha" release state, I know many people
are daily driving it, in various configuations, and reception is generally
positive, and I do appreciate the folks that are running it, and the
encouragement from those same folks.

Today, as of 2023-04-01, MicroOS Desktop Plasma is shipping version 5.27.3,
right in line with Tumbleweed.

Just by way of explanation, I have been testing and developing primarily on
a Desktop machine up to this point (Ryzen 5 5600, RX6700XT, and a couple of
1440p Displays, fairly vanilla stuff), I've been daily driving plasma-wayland,
as the maintainer.   So recently, I have switched my workstation to using a
laptop, with a usb-c dock (Thinkpad T16, Ryzen 7 Pro 6850U, Radeon aGPU, Lenovo
Universal USB-C Dock, same displays), Hardly exotic, but having a dock in the
mix does add a wrinkle or two.

In addition, I did recently get a NAS built up and tossed on the network, which
is sharing things on the LAN, mostly using samba (this will be important later)

**I want to preface the following commentary, by saying this is not a Ragequit,
plasma is garbage, everything sucks post.  I'm currently the most active
maintainer of microOS Desktop Plasma, and a long time KDE user and sometime
contributor.   I absolutely appreciate the effort that goes in both upstream,
and from the openSUSE KDE team, and do not intend any of this to be
disparaging those efforts.**

## The problems

With my new hardware setup, I grabbed the latest snapshot, and installed
microOSD Plasma, without being connected to any kind of external peripherals.
Everything installed, and worked as expected, I reinstalled the flatpaks I
regularly use, and rsynced the stuff off my NAS back into my $HOME.

And then I shut it down, and hooked it up to the dock.   And the wheels
literally fell off.

1. Everything boots up just fine, and I'm presented with the sddm user
selection/login on the laptop screen, and the sddm background on the dual
external displays.   As I said, I'm using plasma-wayland as my default session,
but at that point, the external keyboard and mouse are plugged into the dock are
non-responsive.   I figure, as it's never been booted with the dock connected
before, there's probably just some autosetup stuff that hasn't run before, so I
login on the laptop keyboard, and as soon as plasma starts loading, both
external displays go blank, and give me the OSD for losing signal, and the
keyboard and mouse stay non-responsive.  I unplug the usb-c cable from the
laptop, and plug it back in.  The keyboard and mouse do come to life, after what
seems like 30 seconds, but still no external displays.

2. I reboot, with the dock connected, no external keyboard/mouse available in
sddm, manually select Plasma-X11 session, and login with the laptop keyboard.
I now have the external displays coming up with plasma, but still dead external
kb/mouse.  I unplug the kb/mouse from the dock, and plug them back in, and they
both start working.

3. I disconnect the laptop from the dock, and take it to the couch with me, and
attempt to login from the screenlocker, and the laptop keyboard is
non-responsive, I'm able to use the touchpad to use the OSK to login.

4. Running plasma, in X11, I get *very* regular screen flicking, on both of the
external displays.

5. Regardless of everything I've tried, the external mouse and keyboard do not
work through the dock, in sddm, or plasma, unless I unplug them from the dock,
or unplug the dock from the laptop, and reconnect them, once I get to the plasma
desktop.

6. This one has nothing to do with my new hardware, but my new NAS.   My media
library lives on the NAS, since setting it up.  Things are shared out over smb
shares for the most part, as my wife uses Windows (just save the groaning, or
suggestions I make her switch, after 12 years together it's not happening),
so I want to mount my media share.  I can see the samba server just fine
with dolphin, I can access the share just fine, as a network location.  What
I *can't* do is mount that share anywhere, in dolphin.

## Summary

Basically, microOS Desktop Plasma is absolutely *Alpha* state on my current
hardware.   Yes, everything basically works, I can get things done on it, but
it is hardly optimal.   I've been poking at it, and tweaking things for a few
weeks now, and it isn't getting any better.  I will be the first one to admit
that I'm not a coder, I can muddle my way through bugfixes, modifying
configurations, building packages, maintaining patches, etc.   And the solutions
to some of these problems are beyond my current skill level.

That being said, after suffering through a couple weeks of pain with Plasma, I
made the decision to just try and see what would happen, if I installed microOS
Desktop Gnome on the same hardware, and the same configuration.

Quite literally, everything has worked.   The Wayland session sees my external
displays just fine, I have no screen flickering, my mouse and keyboard work, I
can mount a samba share right through nautilus.

And I even found something that I didn't even know was broken, until I set my
personal e-mail up in Evolution, and connected to my mail host, and all of a
sudden, I've got folders showing up from that IMAP host, that the kontact/kmail
flatpak can't even see, for one reason or another

I've had exactly one "glitch" in the few days so far that I've been on gnome,
that was likely self-inflicted, where I disconnected the usb-c while the laptop
was trying to go to sleep, and it went into a weird locked state.   A forced
shutdown and reboot fixed it.

## Conclusion

I am stating, right now, for those of you that are clamoring for it to be so,
or asking when it will be "release ready" that microOS Desktop Plasma, is not,
and will not be "release ready" anytime soon.

What is holding it back, you ask?
What can I do to help?

1. Time.  Literally this.  The KDE project is somewhat behind on their flatpak
   effort, in comparison to GNOME.  They're working on it diligently, but it's
not a small thing to do, to get the *entire* KDE Software Collection into a new
packaging format, and finding the spots where portals have to be tweaked, etc.

   So if you're the sort that is interested in flatpaks, and learning how to
   work with them, I *highly* suggest heading over to the KDE flatpak guide
   [develop.kde.org](https://develop.kde.org/docs/packaging/flatpak).

2. Some of my personal issue, with this hardware, I *suspect* are due to the
   current sddm not being wayland compatible.  When booting right now, to use
wayland, you're getting sddm as an X11 session, and plasma as a wayland session.
There is wayland support upstream, in some form, but it is not currently
released. I'm not interested in having microOS Plasma shipping a different
version of sddm, than Tumbleweed does.

   So what can be done here?  If you're a coder that can help, sddm is
   developed at [sddm](https://github.com/sddm/sddm), go see if you can't help
   out.

3. Feature parity with GNOME.   Its what it says on the tin.  The only "easy"
   way I sorted out how to mount a smb share in userspace (just to use an
example) was to use t-u to install the gvfs components necessary, so that I was
able to do a `gio mount smb://host/share`.   That still doesn't give any
sort of way within dolphin itself to mount them.   And honestly, that took
me most of an afternoon, bouncing around various places on the web to figure out
how to do.

   If our target is users that want a "install it and go to work" system (and
   it is.), this sort of thing just isn't going to be acceptable.   I do *not*
   know what the official stance from the dolphin or KDE developers is on this
   one, but my websearching didn't turn up much at all about such things.

4. Relying on me, to continue to do this basically by myself.  I am being quite
   frank with you, as the users.  I am *not* the guy that is going to handcraft
a linux distribution from scratch, solve all the problems, squash all the bugs,
and everything else.

   I absolutely do appreciate those of you that have been talking up MicroOSD
   Plasma in the various communication channels, but I need more help than that.

   I'm not trying to make anybody feel guilty, we've all got lives, and it's
   not like I'm being paid for this.

   That being said, I would rather see the plasma version of microOS Desktop go
   away, than be pushed to release in the state it's in, with some vague hope
   that the problems are going to be fixed.

   For this to have *any* chance of getting past anything better than a "Beta",
   I need *real* help, people pushing SR's, people actually reporting bugs
   properly on the bugzilla, etc.

   **Reddit is not a bugtracker.
   Matrix is not a bugtracker.
   IRC is not a bugtracker.**

   Yes, the bugzilla can be a little clunky, but it's the tool we've got. I
   don't have the time, or the inclination to be constantly monitoring things
   like Reddit/Matrix, nor should anybody else feel like they need to.

   It seems like there are lots of folks that feel strongly that MicroOS
   Desktop Plasma needs to exist.   But so far, I've seen darned little actual
   "put your money where your mouth is" and that is a problem, with a project
   like this. We are a community distribution, run by volunteers, and without
   people contributing their time and energy to do the stuff that isn't all that
   fun, it just isn't going to happen, no matter how many end users want it to.

Long and short of it: my daily driver is actually going to be GNOME for right
now, because I actually need to use my computer. Right now, microOS Desktop
Plasma is putting obstacles in my way.   I do have another machine here that
still has Plasma on it, for testing things on actual hardware. I'm not
throwing my hands up, grabbing my ball, and going home. I also don't want to
to be giving any false hope that I'm going to magically wake up tomorrow and
pull the proverbial magic rabbit out of my hat.
