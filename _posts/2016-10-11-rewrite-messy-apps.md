---
title: How to Refactor Messy Apps for a New Architecture?
created_at: 2016-10-11 16:44:00 +0200
kind: worklog
tags: [ layered-architecture, software-architecture, refactoring ]
comments: on
---

When facing a legacy code base, changing the mess to an orderly architecture can cause confusion: where to start? What to do? An exemplary question:

> How do I refactor a Big Ball of Mud into layered architecture?

Refactoring[^r] the existing code base seems like a logical step; after all, refactorings are designed to improve existing code, and improvement is what you're up to.

[^r]: This is only a refactoring on the app level, as in "improving its code," but not in the usual meaning: "Refactoring is a disciplined technique for restructuring an existing body of code, altering its internal structure without changing its external behavior." (<http://www.refactoring.com/>) I'll go with the colloquial use of the word here anyway.

But:

* Changing the existing mess is a confusing process because your **mental model** from deciphering the legacy code gets in the way. You cannot interpret existing code one minute _and_ come up with great new designs independent from the mess you have the next.
* Seemingly clear cases get in the way: moving a view controller sub-type into the UI layer sounds trivial. It's misleading, though, since most view controllers are a mess of responsibilities. So filing away the object as a first step will not be helping at all.

My advice:

Create a new model of the application. Start from scratch, and start with drawings on paper. This is an exercise to _understand_ the problem and solution space in a new way. Then you may decide if your legacy code can be re-used to fit the better architecture or if you have to rewrite parts of it. If you look at the old stuff all the time, you can only force yourself to make incremental improvements, not re-designs.

It helps to factor the new model into modules and submodules in advance to start somewhere. Instead of stopping at 4 layers in your app with a bunch of objects in them, identify submodules and communication paths, too, so you can hunt for a small-ish part of the app that you can improve.

To learn how to apply a particular architecture, it's better to start with a greenfield app or a toy project so you can get a hold of the concepts first without worrying about breaking a shipping app. With experience, it'll be easier to identify new seams in old code bases and change things for the better.
