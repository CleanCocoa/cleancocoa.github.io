---
title: NSSplitViewItem Vibrancy Is Not Added with Every Initializer
created_at: 2017-06-18 09:27:11 +0200
tags: [ nssplitview ]
comments: on
---

After yesterday's [wrap up of my fix for vibrant table views](http://cleancocoa.com/posts/2017/06/nssplitviewcontroller-visual-effects/), I have discovered that `NSSplitViewItem` has 3 initializer variants since macOS 10.11:

* `init(contentListWithViewController: NSViewController)`
* `init(sidebarWithViewController: NSViewController)`
* `init(viewController: NSViewController)`

The latter will not add vibrancy. Guess who, by accident, discovered his table view to be wrapped in a `init(sidebarWithViewController:)` call.

I'll leave the other post up as reference for deactivating vibrancy and dealing with `NSTableView` text drawing intricacies.

