---
layout: default
title: Adventures with battery life, powertop and breaking wifi.
---
I wasn't happy with the battery life of my little lenovo s-21e. Previously on Windows 8.1 I'd get as much as 8 hrs of battery life. WIth arch installed, using a light i3 window manager, this was a paltry 2 hours. Unacceptable

The arch wiki revealed that installing the tlp package and using PowerTOP to optimize the system could help. After running

    powertop --calibrate

and

    powertop --autotune

this was improved to a solid 7.5 hours.

I'll take it.

Just one problem. No wifi! A reboot didn't fix it. Running wifi-menu would just return "no networks found". I was pretty sure that the device was still working as it showed up in 

    ip link

so the problem had to be software related. *Wifi-menu* is just a wrapper with a little ncurses ui, so I decided to go to the most basic way of starting up wifi networking.

    ip link set *wifiadapter* up

Bingo! This returned an error that the network was unable to start because RFkill was blocking the wifi adapter.

Turns out PowerTOP will do a soft wifi block to try and save battery life. Wat. Seems pointless. I guess if you're on an airplane it could be handy. I should do more reading on the functionaly of powertop as well. Maybe I missed something while running it.

ANYWAYS, \*deep breath\*, simply running

    rfkill unblock wifi

fixed the issues. Seems to hold up over a reboot too.
