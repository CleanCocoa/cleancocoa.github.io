---
title: "5 Heuristics for \"I have a complicated nested view controller setup. How do I handle passing data?\""
created_at: 2016-03-25 13:23:52 +0100
kind: worklog
tags: [ mvvm, viper, view-controller, flow ]
vgwort: http://vg01.met.vgwort.de/na/62e5ed85a39a418482e064e5dc0942d9
comments: on
---

That's a [recent question](/posts/2015/06/start-mvvm/#comment-2578197614) from the comments put in my own words. A **view model** that encapsulates the display state sounds promising at first. But when you don't have a simple view that displays one thing, how do you model that in code? How do you model a sequence of view controllers, for example a more complex checkout process?

There are a few heuristics I adhere to for as long as humanly possible (i.e. there are cases where it just doesn't work, but I try to neglect this for as long as I can):

* **The view must be dumb.** Create `UITableCellView` subclasses that display a view model's data in themselves. Create `UIViewControllers` (yes, that's part of the view, too!r that manage their components' lifecycle. But don't try to put _any other logic_ in there. Pass button presses around to another layer. Don't add the slightest meaningful reaction. The most a view component is allowed to do in reaction to user interaction is fire off a notification.
* **Create an application service layer.** Call it what you will: Core Data/iCloud background synchronization doesn't belong into the view, so it has to go somewhere else. The `AppDelegate` is the next but also the worst place to put this, because then it'll grow and do everything that's not view-related. Create a layer below the user interface layer. (I adhere to the tradition of calling it "Application layer".) Create a `CoreDataBackgroundSynchronizer` in there. In fact, I prefer to come up with action-oriented names to keep this focused, like `PullUpdatesFromServer`. If something's not related to _pulling_ data, like merging data, delegate the pulled stuff to a `MergeServerData` object. And so on.
* **Separate read from write model.** A `BananaViewModel` contains everything you need to display a precious banana on screen, including `UIImage` properties. If a user is allowed to edit banana details, make your life simple and model the "edit" path in a different way. `NewBanana` or `ModifiedBanana` work very well as struct names. Then you can focus on how saving changes work in one type and don't have to worry about inadvertently ruining display code.
* **Create facades.** If a `UIView` subclass in a `UITableCellView` in a `UIViewController` contains a `UIButton` that is tapped, how will the button press reach the event handlers that reside in the above-mentioned Application layer? Through the topmost view object you own. The `UIViewController` can expose setting a callback block that, when set to something, passes this down to the cell which passes it down to its sub-view. The view controller then becomes the facade for the interface. Other layers don't need to have intimate knowledge about the intricacies of your view's setup. (And they shouldn't, because, tight coupling, you know?)
* **Favor creating more types.** This takes some getting used to. You might have questioned my sanity when I mentioned creating a `PullUpdatesFromServer` _and_ a `MergeServerData` service. "Two extra objects for essentially one task?!" -- Only these are two distinct tasks. Just like functions have natural separations. _Divide and conquer,_ as my teacher used to say. Makes unit testing easier, too, by the way, which is overall useful feedback. Create more service objects if you need, create more state-representations if you need. Does your mutable order process state contain user info, basket items, and payment info? Create one type for each, make one view controller responsible for creating instances of each of them, and then have an overarching `OrderProcess` model objects that holds all of these together.

These heuristics will not magically cure your problems. But these imperatives have helped me make a decision that didn't target short-term benefits (a.k.a. "quick fix") and they may help you, too.

You might also like to read about the [SOLID](https://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) object-oriented design principles to give this another angle.
