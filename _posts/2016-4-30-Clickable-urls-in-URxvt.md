---
layout: default
title: Clickable URLs in URxvt
---
Been hanging out in IRC (irssi) and not being able to easily click or grab memes linked in chat was bothering me. I did some googling and found out that if I added options to enable perl extentions to the .Xresources file and set a default browser (or xdg-open) to open links, I could now rapidly keep up with the maymays!

    URxvt.perl-ext-common:      default,clipboard,url-select,keyboard-select
    URxvt.url-select.launcher:  firefox
    URxvt.matcher.button:       1
    URxvt.url-select.underline: true
    URxvt.keysym.M-u:           perl:url-select:select_next
    URxvt.keysym.M-Escape:      perl:keyboard-select:activate
    URxvt.keysym.M-s:           perl:keyboard-select:search
