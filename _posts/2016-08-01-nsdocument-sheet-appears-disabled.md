---
title: "NSDocument Weirdness: Sheet Contents Appear Disabled"
created_at: 2016-08-01 16:14:15 +0200
kind: worklog
tags: [ appkit, nsdocument, dialog ]
comments: on
---

When I added a sheet to display on top of TableFlips' document, I wondered why the text field appear disabled, tabbing through elements didn't work, and overall functionality was limited to accepting click events:

{% include figure.md src="/assets/blog/2016/disabled.png" alt="screenshot of disabled sheet" caption="Everything appears disabled in this sheet" %}

It turned out you have to make sure that you disable most of the `NSWindow` settings in Interface Builder _except_ the title bar (`NSTitledWindowMask`). Only with a title bar (which is never visible in a sheet anyway) will the interaction work properly.

After ticking that checkbox, everybody's happy:

{% include figure.md src="/assets/blog/2016/enabled.png" alt="screenshot of working sheet" caption="Element focus works properly, including tabbing around and canceling with ESC" %}

Sometimes, AppKit makes me crazy.
