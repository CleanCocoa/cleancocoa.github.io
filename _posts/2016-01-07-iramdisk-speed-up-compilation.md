---
title: iRamDisk May Speed Up Your Xcode Compilation Times
created_at: 2016-01-07 18:23:19 +0100
kind: worklog
tags: [ xcode, performance ]
image: "iramdisk_512.png"
comments: on
preview: fulltext
---


I code on a late 2011 Mac Mini with 8 GB RAM and a 256 GB SSD. This machine is a ton faster than my old MacBook Air was. Compilation still takes time, but it seems that [iRamDisk][mas] helps a bit.

Lately I wondered if I could cut down the 30s compilation time (3mins with a clean build folder) if I had a Mac Pro, or MacBook Pro, or iMac, or whatever next tier device. Faster cores, more cores, doesn't matter. But buying a Mac for $3k is out of question at the moment. So I've been looking for other tricks.

The only thing that's faster than a SSD is RAM. That's when I [found](https://paulofierro.com/blog/2014/3/16/speeding-up-xcode-development) the app [iRamDisk][mas]. It offers an option to move the derived data folder to a virtual disk in RAM. It seems I can reduce the clean build time from 3mins to 2mins with that. Xcode needs about 1.5GB derived data for the Word Counter -- at least that's how big I had to make the RAM disk to not get filled quickly. The overall memory pressure is a bit higher, the Mac is using swap more, but compilation becomes faster at last.

There's a downside, too: after every reboot the RAM disk is empty, so Xcode has to index the project again and compile fresh at least once a day. Depending on your usage this can be annoying or not be a problem at all.

You can download a trial from [the website](http://www.magnesium-app.com) or [buy it for $19.99 on the Mac App Store.][mas]

[mas]: https://itunes.apple.com/us/app/iramdisk/id492615400?mt=8&uo=4&at=11lxCd
