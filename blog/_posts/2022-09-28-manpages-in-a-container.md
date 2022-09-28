---
layout: post
title:  "Manpages as html in a container"
date:   2022-09-28 11:00:00 +0200
author: Thorsten Kukuk
---

## Intro

openSUSE MicroOS has one deficit since the beginning: the documentation will not be installed on the system.

For standard commands this is no problem: you can read the manpages on Tumbleweed, Leap, ... or other distributions or web pages.
But for some MicroOS specific tools this is no option.

Back in the "good old times" some decades ago Sun had something I really
liked: a documentation server serving all documentation, which you could
also install on the local network. This is what I always wanted to have
for openSUSE, too :)

So wouldn't it be nice to have all documentation accessible from your webbroser?

## Reading manpages with a webbrowser

To read manual pages and other documentation in a webbrowser, you need a web server serving the data. Since openSUSE MicroOS is a Container Host OS, it's quite logical to put that into a container.

Now here it is, start your own MicroOS documentation server:

```
podman run -it --rm --name docserv -p 80:80 -p 443:443 opensuse/microos-docserv
```

and connect as `http://localhost` or `https://localhost` with your
webbrowser to browse, search and read our manual pages.

## Next goals

The next goal is, to not only provide a container with the manual pages
for MicroOS, but something like "manpages.opensuse.org" containing all
manpages from openSUSE Tumbleweed!

But for this help is needed:

1. Working style sheet
Alexandre was so nice and created something for the start. But this
needs more testing and bug fixing, especially the mobile support.

2. Generate somehow the data for the webpages
We need access to a full tree of openSUSE Tumbleweed, generate the data
and upload it to a webserver.

3. manpages.opensuse.org
We need that sub-domain, and a machine serving the data.

4. Integrate additional documentation
How can we integrate other docu? texinfo? HTML docu?

Every help is welcome and needed for testing, development, integration.

The github repo is [rpm2docserv](https://github.com/thkukuk/rpm2docserv).
Packages and the container description are already part of Tumbleweed.

The whole idea and parts of the code are based on the [debiman](https://github.com/Debian/debiman) project.
