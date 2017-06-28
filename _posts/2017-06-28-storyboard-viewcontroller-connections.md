---
title: macOS Storyboard Outlet-Like Connections Between View Controllers
created_at: 2017-06-28 13:06:42 +0200
tags: [ storyboard, mac ]
vgwort: http://vg01.met.vgwort.de/na/b8ff828259484f16806f34d3e99740f5
comments: on
---

Transitioning from Nibs to Storyboards poses a few challenges. One of them being creating outlets from a parent view controller to a child view controller. There is no such thing.

You could say this is the new idiomatic way to create interfaces, but I don't quite agree with the consequences. How do you pass data to a view controller 4 levels deep in the hierarchy? It's not obvious. And there are no "embed" segues like you have them on iOS [to obtain references](https://stackoverflow.com/questions/8198698/linking-child-view-controllers-to-a-parent-view-controller-within-storyboard).

Just as I have [beef with Segue's](/posts/2015/01/02/segues-vs-tell-dont-ask/), I don't want my view controllers to be the central point of control. I want to tell them what to do. Eventually, I guess I'll have to settle for a programmatic view, but that day is not today.

Turns out that `NSViewController` now sports a `childViewControllers: [NSViewController]` property which you can use to query for parent--child-relations.

As usual, my custom `NSWindowController` subclass also was the façade for the whole view hierarchy, implementing all view-related protocols to forward the `display(foo:)` commands to the child view controllers. That made the transition from Nib to Storyboard very easy, in fact, since there's just a single point of change. High locality of change for the win! -- If only the outdated child view controller outlets were easy to replace.

The best way I came up with is, upon `windowDidLoad`, traverse the view controller hierarchy and stop when a good match is found. Since even complex views won't have hierarchies hundreds of levels deep, this doesn't even take a noticeable amount of time.

This helper function looks for a match by type in a given view controller's child collection, recursively:

```swift
func firstChildViewController<T: NSViewController>(
    _ parentViewController: NSViewController
    ) -> T? 
{
    for viewController in parentViewController.childViewControllers {
        if let match = viewController as? T { return match }
        if let subChild: T = firstChildViewController(viewController) { return subChild }
    }

    return nil
}
```

Since it's generic, you just have to specify the desired type to look for so the compiler can figure out what you want:

```swift
let bananaVC: BananaViewController 
    = firstChildViewController(jungleViewController)
```

The only limitation is that this will stop on the first match. If you use the same type of `NSViewController` subclasses in multiple places, you to check for an additional criterion.

Inside my window controller, I use this function as follows:

```swift
class AmazingWindowController: NSWindowController {
    var bananaViewController: BananaViewController!
    var appleViewController: AppleViewController!
    
    override func windowDidLoad() {
        loadChildViewControllers()
        // ...
    }
    
    fileprivate func loadChildViewControllers() {
        self.bananaViewController = getChildViewController()
        self.appleViewController = getChildViewController()
    }
    
    fileprivate func getChildViewController<T: NSViewController>() -> T! {
        let firstViewController = self.window!.contentViewController!
        return firstChildViewController(firstViewController)
    }
}
```

Okay, so `getChildViewController` sounds like we're back in Java-land. But it's the best name I could come up with; "load" doesn't fit since the view controller already is loaded. "Fetch" may work, or "find". Whatever suits you.

With this in place, I am able to instantiate the child view controller reference properties in a fashion that resembles `@IBOutlet`s closest.

And this, in turn, paves the way to extract the façade functionality from the window controller into another object. Nice!
