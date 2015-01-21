---
layout: post
title: "ALSA: switch audio output to USB headset"
date: 2014-09-17 13:31
comments: true
categories: [debian, alsa, linux, os]
---

I purchased USB wireless headset, exactly this:
http://www.amazon.it/gp/product/B005FY61MM . It is very comfortable and works
out-of-the-box on my Debian, without any driver.  
Now I have this problem: I want that when I plug the USB receiver automaticaly
the ALSA output switch from the integrated sound card to the headset. This can
be done with _udev_ rules (udev is a device manager for the Linux kernel).  
Open the file `/etc/udev/rules.d/00-local.rules`, if not exists create it; and
add this rules:

    # Set USB headset as default sound card when plugged in
    KERNEL=="pcmC[D0-9cp]*", ACTION=="add", PROGRAM="/bin/sh -c 'K=%k; K=$${K#pcmC}; K=$${K%%D*}; echo defaults.ctl.card $$K > /etc/asound.conf; echo defaults.pcm.card $$K >>/etc/asound.conf'"

    # Restore default sound card when USB headset unplugged
    KERNEL=="pcmC[D0-9cp]*", ACTION=="remove", PROGRAM="/bin/sh -c 'echo defaults.ctl.card 0 > /etc/asound.conf; echo defaults.pcm.card 0 >>/etc/asound.conf'"

In practice when the headset USB receiver is plugged, udev detect it and run a
script to change the file `/etc/asound.conf` setting the ALSA output device with
the headset.  
The only flaw is that when the ALSA output change, all programs that are using
ALSA must be rebooted. If you find a way to solve this problem ping me please.
