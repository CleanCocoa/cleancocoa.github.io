---
title: How to Change the Detail of a UISplitViewController in the Background
created_at: 2015-09-16 10:24:41 +0200
kind: worklog
tags: [ uisplitviewcontroller, segue, storyboard ]
comments: on
---

A `UISplitViewController` has a master and a detail view controller. When the selected item is deleted and the detail is visible, you can perform a segue. If the detail is not visible, performing a segue to change the detail will present it.

I didn't want that.

There are two cases where you will want to change the detail although it's not visible:

* your app runs on an iPhone 6 Plus and was rotated to portrait, thus the detail is now hidden and a segue will result in a weird and unnecessary transition, or
* you support Slide Over and Split View for iOS 9 and iPad Air 2's, where the size of your app's window may change.

Both have in common that the horizontal size class can become "compact" (`UIUserInterfaceSizeClassCompact`) and that this isn't fixed per device anymore but may change over time.

Let's say you invoke `-displayNoSelectionDetailScene` from your master's `-tableView:commitEditingStyle:forRowAtIndexPath:`, that is when the user deletes an item from the table view. If the deleted item is the same as the currently selected one, you have to "close" the detail view in order to not present stale data.

So here is my `displayNoSelectionDetailScene` implementation:

    #!objc
    - (void)displayNoSelectionDetailScene
    {
        if (!self.splitViewController.collapsed) {
            
            // Show empty selection detail only when the detail is visible.
            [self performSegueWithIdentifier:kDetailSegueNoSelection sender:self];
        } else {

            UINavigationController *emptyNC = [self.storyboard 
                instantiateViewControllerWithIdentifier:kViewControllerNoSelection];
                
            // I'm so used to unwrapping nil that I get paranoid ...
            assert(emptyNC != nil);  
        
            self.splitViewController.viewControllers = 
                @[self.splitViewController.viewControllers.firstObject, emptyNC];
        }
    }

If the `UISplitViewController` is collapsed, the view controller or the whole app has a compact horizontal size class, for example when the iPhone 6 Plus is rotated to portrait.

At first I thought I should hook in to `-willTransitionToTraitCollection:withTransitionCoordinator:` and reset the detail view controller when the horizontal size class changes -- but the results were weird. Upon every table view selection did the trait collection reported a change from compact to regular:

    #!objc
    - (void)willTransitionToTraitCollection:(UITraitCollection *)newCollection withTransitionCoordinator:(id<UIViewControllerTransitionCoordinator>)coordinator
    {
        if (self.traitCollection.horizontalSizeClass == UIUserInterfaceSizeClassCompact 
            && newCollection.horizontalSizeClass == UIUserInterfaceSizeClassRegular) {
        
            // Dismiss previously open details in the background
            [self displayNoSelectionDetailScene];
        }
    }

Didn't work. 

Quickly replacing the view controllers of the `UISplitViewController` when in a collapsed state does. 
