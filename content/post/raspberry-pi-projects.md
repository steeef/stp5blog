+++
date = "2016-04-07T10:12:45-07:00"
draft = true
tags = ["raspberry-pi", "pi-hole", "retropie"]
title = "raspberry pi projects"

+++

# Interesting Raspberry Pi Projects

I have a few Raspberry Pi Model B's laying around, and I've been meaning to
try out some things with them. I also bought the recently-released [Raspberry
Pi 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) that I wanted
to find a use for.

## Pi-hole

[Pi-hole](https://pi-hole.net/), put simply, is an ad-blocking, anti-tracking
DNS server that you can use in your home network. It was super-easy to set
up, and even has a pretty web interface for viewing stats:

![Pi-hole web interface](/blog/images/posts/pihole-web.png)

Included is a script that sets up everything for you, and schedules a weekly
cron job to update block lists. However, the project website also includes
complete instructions for doing everything manually.

I now have one of my Pis sitting in a cabinet next to my router performing DNS
requests without any issues.

## Retropie

[RetroPie](http://blog.petrockblock.com/retropie/) is a project around enabling
the Raspberry Pi to be used as a video game console emulation station for as
many platforms as possible. As with Pi-hole, they provide a script that sets up
everything for you, and the default settings are adequate for most. You simply
plug it into your display/TV/whatever, copy some games (via a USB drive if
you've got it) and go. There's enough of a community here to get most games
working accurately.

I installed RetroPie on my Pi 3 and it worked like a charm. I plugged in
a couple XBox 360 controllers and started playing old SNES and Playstation
games without too much trouble. Now I just need to be patient until my son's
old enough to play with me...

## Other ideas

I don't have a lot of products I'd consider in the Home Automation category,
but it's inevitable that we'll have more things available at home that could be
controlled with a central hub. [Home Assistant](https://home-assistant.io/) is
a Python project around creating a full-featured hub for this purpose. Looks
neat!

More sites for Raspberry Pi project examples and ideas:

* https://hackaday.io/projects/tag/raspberry%20pi
* http://raspberry.io/projects/
* https://www.hackster.io/raspberry-pi
