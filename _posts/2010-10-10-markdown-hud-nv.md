---
title: Markdown HUD for Notational Velocity
created_at: 2010-10-10 22:40 +0200
image: "hud.png"
kind: worklog
tags: [nv, markdown, mac]
---

{% include figure.md src="/assets/blog/2010/hud.png" alt="screenshot of NV" %}

This picture shows my favorite style of previewing notes with [MultiMarkdown][mmd] in [Notational Velocity][nv]: in a separate tool window which floats on top.  I chose this so-called HUD design ("heads-up display", like a display made of glass in aircrafts) which most of you will already be familiar with by using Apple's _Quick Look_.

The preview window is invoked by pressing **&#8984;&#8963;P** (Cmd-Ctrl-P) or by using the new _Preview_ menu.  There, you'll be able to switch between three currently supported rendering modes:  Textile, classic Markdown and MultiMarkdown.  But now, these menu items are dysfunctional.

Another idea to make the preview less obstrusive is [using a drawer which slides out of NV's main window][pe].  Since implementing this new visual representation was _so unbelievably easy_ (remember I'm new to Mac development), changing it again should not be too much of a problem.

What do you think?  Any suggestions? Mail me or reply in twitter! (@ctietze)

[nv]: http://notational.net/
[mmd]: http://fletcherpenney.net/multimarkdown/
[pe]: http://www.practicallyefficient.com/2010/09/29/a-call-for-folks-interested-in-working-on-notational-velocity/

## Go get it ##

*   [Project Website with binary downloads](/zettelkasten/nv)
*   [GitHub code repository](http://github.com/DivineDominion/nv)
