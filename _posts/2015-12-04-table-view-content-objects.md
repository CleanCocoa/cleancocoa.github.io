---
title: Extract UITableView Contents and Configuration Into Helpers
created_at: 2015-12-04 21:11:43 +0100
kind: worklog
tags: [ uitableview, view-controller, model ]
comments: on
---

In a top secret project I am working on, I think I found a consistent way to create `UITableViewController`s in a reusable fashion without making a mess with [massive view controllers][1].

The key ingredient is, as always, delegation from the view controller and thus composing larger components:

A `BananaViewController` ...

* ... is a table view controller.
* ... consumes `BananaViewModel`s on its `showBanana()` method.
* ... delegates displaying a huge header to `BananaHeaderView`.
* ... delegates cell display to `BananaViewControllerContents`, which is data source and delegate to the table view.
* ... responds to and delegates upward events from its contents.

The table view controller mostly delegates things away. It orchestrates the setup and keeps a strong reference to its components. It does't change itself _at all_ when new data is displayed, though. 

This is huge, because you can now explicitly model view controller state changes as messages sent from the cells, for example.

The usual approach in contrast is to change the view controllers state upon interaction somehow. This will work in simple apps and with manual testing kind of fine, but it doesn't scale well. Unit testing is very hard, too, because everything is so intertwined.

A better approach will utilize mutation of a model object instead which is the [source of truth](/posts/2015/07/refactor-information-flow/) for the table view's data source methods. The natural next step is to pry out that logic from the view controller, too, and then extract cell setup. Tests for these simple collaborators are short and highly focused. 

The basic pattern is "copy & paste reusable." If this works out well, I may even create abstractions, but I doubt this will be worth the effort for a while. Copy & paste works well enough for moving quickly at this early stage of development.

So that's how data display flows really nicely down the tree of components. User interaction poses a different challenge: how are messages passed up the chain again? I'm experimenting with a few different approaches and will keep you posted once I settle for a particular solution.

[1]: /posts/2015/11/massive-view-controller-zarra/
