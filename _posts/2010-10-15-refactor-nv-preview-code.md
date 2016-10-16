---
title: How did Steve add Markdown to Notational Velocity?
created_at: 2010-10-15 12:20 +0200
kind: worklog
tags: [nv, markdown]
---

I found out [what Steve did to add Markdown preview](http://github.com/panicsteve/nv/commit/ce6bf6e5cc3a635ed51fbecffd486ee97808220e).  There are a few additions to [Notational Velocity](http://notational.net)'s graphical interface components, i.e. the third pane in Steven's case, the HUD in mine respectively.  The other changes are easily reproduced and surprisingly few.

Why do I mention this?

Well, Zachary, the author of Notational Velocity, continues to improve his software.  Only problem here is Steve not being up to date so much.  I wanted to integrate the new fixes Zachary introduced, but failed.  It looks like it's pretty easy to change existing software a little bit to add something it previously lacked to fulfill a certain task.  On the other hand, _intertwining_ two different things is probably not the best way to solve an issue in terms of upgrades to the original piece.

What I'm going to do is separate the new "logic" to preview (Multi)Markdown and Textile from the original source in a way which enables me to incorporate future changes from Zachary into my modified version and makes it easier for [other developers](http://elasticthreads.tumblr.com/nv) to merge my preview with their additions.

When I'm done refactoring code, all your helpful feedback, ideas and reported bugs should be easier to maintain for me.  I really appreciate your support!
