---
title: MVVM's Place in Your App
created_at: 2016-10-08 12:32:56 +0200
kind: worklog
tags: [ mvvm, software-architecture, layered-architecture ]
comments: on
---

Picking up on my post ["MVVM Is Quite Okay at What It Is Supposed to Do"](/posts/2016/08/mvvm-is-okay-for-what-it-does/), here's a few images which illustrate the problem of mistaking MVVM for a solution to a structural problem. It's the whole post in 2 images.

Model--View--View-Model helps with the **view layer**. It can be a tool to break up a view controller into smaller things. But it's still only a refactoring of view components into more objects.

{% include figure.md src="/assets/blog/2016/mvvm-layered.png" alt="diagram of MVVM in the view layer" caption="MVVM in the view layer" %}

That's useful and not a problem itself. But it doesn't help if your overall app is a mess.

The technical term for "mess" on the level of software architecture is "[Big Ball of Mud](https://en.wikipedia.org/wiki/Big_ball_of_mud)"; combine MVVM and BBoM and you have ... MVVM in a BBoM.

{% include figure.md src="/assets/blog/2016/mvvm-bbom.png" alt="diagram of MVVM components in a chaos" caption="MVVM in a BBoM. Kind of rhymes, doesn't it?" %}

There you see that MVVM can affect the picture of your architecture -- but not by much. It's a design pattern you can use to solve a particular coding problem, but it can't bring order (or _structure_) to the chaos.

