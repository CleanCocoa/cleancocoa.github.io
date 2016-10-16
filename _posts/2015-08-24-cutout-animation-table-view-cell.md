---
title: Cut-Out Animation of a Table View Cell Into the Next Scene
created_at: 2015-08-24 12:41:41 +0200
kind: worklog
tags: [ animation ]
url: https://realm.io/news/altconf-marin-todorov-animations/
comments: on
preview: fulltext
---

I haven't dabbled with animation a lot. So I was pretty surprised that there's a method on `UIView` called `snapshotViewAfterScreenUpdate` which creates a static duplicate which you can use for animations easily.

I found this in a AltConf talk by Marin Todorov called ["Power Up Your Animations!"](https://realm.io/news/altconf-marin-todorov-animations/). There, he does ... well, I don't know what it's called, or else I would've picked a better title. See for yourself.

Here's a crappy screen capture of the animation in his talk in action:

{% include figure.md src="/assets/blog/2015/201508191421_view-copy-anim.gif" alt="animation" caption="Visually cut-out the cell by copying it, then animate a scene transition while it hovers on top, then move it to top." %}

His [code is on GitHub](https://github.com/icanzilb/PowerUpYourAnimations/blob/master/PowerUpYourAnimations/TableCellAnimator.swift), where you can check out how the `TableCellAnimatior` works. I think this is a very amazing, yet subtle animation. I find it unbelievable how easy it is to implement. 

It seems I have no reason to fear animation in the future.
