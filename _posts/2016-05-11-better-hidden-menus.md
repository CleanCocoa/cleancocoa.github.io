---
title: Better Sliding Menus
created_at: 2016-05-11 16:27:40 +0200
kind: worklog
tags: [ ux, menu, uisplitviewcontroller ]
vgwort: http://vg01.met.vgwort.de/na/ee65ee2391384280b2048305f1048255
comments: on
---

In a recent [usability post of Raluca Budiu][men], the common mobile app pattern of sliding menus is criticized: when the menu button is at the left edge and the content slides away to the right, then hiding the menu again requires the user to move her hand, and reading the menu items requires her to move her finger out of the way. With fat fingers, this is even more of a problem.

Here's two conceptual images with an overlaid fat finger silhouette so we're on the same page:

{% include figure.md src="/assets/blog/2016/sliding.jpg" alt="screenshot of sliding menus" caption="When the user reveals the hidden menu, the menu toggle moves away so she has to reach with her thumb. Or, worse, use the opposite hand to tap the button." %}

I wonder why people favor this fancy pattern where the content slides to the right and reveals a menu to a menu that slides out like the good old pull-down menu or your well-known web site hover menu.

{% include figure.md src="/assets/blog/2016/overlay.jpg" alt="screenshot of overlaying menu" caption="The menu expands below the user's finger" %}

Menu items should be aligned to the right of the toggle so the finger doesn't obstruct scanning of the menu. 

It's like a generic `UISplitViewController` on an iPad in portrait behaves. There, the master view controller overlays the detail view controller by default. The only but significant difference is that the toggle stays in place.

As a first step towards this goal, using a `UISplitViewController` plus adding a separate toggle to the master view controller can do the trick already.

[men]: https://www.nngroup.com/articles/expandable-menus/
