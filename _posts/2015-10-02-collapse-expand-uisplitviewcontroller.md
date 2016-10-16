---
title: How I Now Deal with Collapsible Split View Controllers on the iPhone 6
created_at: 2015-10-02 12:58:02 +0200
kind: worklog
tags: [ uisplitviewcontroller, clean ]
comments: on
image: 201510021420-empty.png
---

So `UISplitViewController` is the new default for iOS apps. Most iPhones have compact-sized size classes. iPads have regular sized ones. The iPhone 6(s)&nbsp;Plus mixes both. Only horizontally regular-sized environments show both the master and the detail scene (or primary and secondary view controller).

When you rotate from portrait to landscape, the "Plus" phones split the screen's contents like iPads do. That's because they are not of compact but of regular height. When rotating, this translates to regular width.

So depending on the device orientation, the screen will have one compact and one regular-sized dimension.

Now UIKit provides two methods to deal with this switch, called collapsing and expanding:

    #!objc
    - (BOOL)splitViewController:(UISplitViewController *)splitViewController 
    collapseSecondaryViewController:(UIViewController *)secondaryViewController 
          ontoPrimaryViewController:(UIViewController *)primaryViewController;

    - (UIViewController *)splitViewController:(UISplitViewController *)splitViewController 
    separateSecondaryViewControllerFromPrimaryViewController:(UIViewController *)primaryViewController;

Handling collapsing is pretty easy. You return `YES` or `NO`. The three parameters give you plenty of context to make an informed decision.

Note that I let the master view controller implement `UISplitViewControllerDelegate` to provide functionality.

For Calendar Paste 3, it looks like this:

    #!objc
    - (BOOL)splitViewController:(UISplitViewController *)splitViewController collapseSecondaryViewController:(UIViewController *)secondaryViewController ontoPrimaryViewController:(UIViewController *)primaryViewController
    {
        if ([secondaryViewController.restorationIdentifier isEqualToString:kViewControllerNoSelection])
        {
            // Always collapse empty detail
            return YES;
        }
       
        // Don't collapse paste detail
        return NO;
    }
    
There's a "No selection" detail scene when rotating to landscape. It's so boring that it should collapse when rotating back to portrait so the table view is visible instead. When there's meaningful content to the user displayed as detail, this content should become visible when rotating into portrait instead of the table view. Easy.

{% include figure.md src="/assets/blog/2015/201510021420-empty.png" alt="Calendar Paste screen shot" caption="Empty detail scene for calendar paste" %}

What should rotating back into landscape do? 

* When the app shows the table view list in portrait, show the empty list as detail in landscape.
* When the app shows another view controller in portrait, i.e. when it was pushed onto the stack, show this one as detail in landscape. 

The separation-callback doesn't tell what the secondary view controller is going to be.

How do you find out which detail view controller is currently visible?

My master view controller, which is the delegate, does this:

    #!objc
    - (UIViewController *)splitViewController:(UISplitViewController *)splitViewController separateSecondaryViewControllerFromPrimaryViewController:(UIViewController *)primaryViewController
    {
        if ([self isDisplayingPasteDetailSceneWithPrimaryViewController:primaryViewController])
        {
            // Keep the detail scene
            return nil;
        }
    
        // Replace the detail scene with the "empty" scene
        return [self emptySelectionNavigationController];;
    }

Apart from the hideous method name `isDisplayingPasteDetailSceneWithPrimaryViewController:`, it's still not too complicated to understand what the logic is.

What does it look like, you ask?

    #!objc

    - (BOOL)isDisplayingPasteDetailSceneWithPrimaryViewController:(UIViewController *)primaryViewController
    {
        if ([primaryViewController isKindOfClass:[UINavigationController class]]) {
            UINavigationController *navController = (UINavigationController *)primaryViewController;
        
            if (navController.viewControllers.count > 1) {
                UINavigationController *detailNavController = (UINavigationController *)navController.viewControllers[1];
                id visibleDetailViewController = detailNavController.viewControllers.firstObject;
            
                if ([visibleDetailViewController isKindOfClass:[ShiftAssignmentViewController class]]) {
                    return true;
                }
            }
        }
    
        return false;
    }

It's a nightmare!

Here, I strongly couple knowledge about implementation details of `UISplitViewController` and `UINavigationController` into the master view controller.

Fun facts:

* In portrait, `UISplitViewController` has just one entry in its `viewControllers` array.
* In landscape, it has two. The second is the detail. That'd help in portrait, too.
* Setting `splitViewController.viewControllers` when in portrait breaks your app. It changes the detail behind the scenes, but at the same time renders segues to any detail scene defunct. (That's the bug of v3.0.0 I had to fix today.)
* In portrait, the master `UINavigationController` knows about the visible detail view controller. After all, the detail is pushed onto the visible navigation stack just as if no `UISplitViewController` was used.

After all it seems `UISplitViewController` does weird things to the master's `UINavigationController` to achieve its magic.

This makes it reasonable for Apple to not provide a `secondaryViewController` parameter in `-splitViewController:separateSecondaryViewControllerFromPrimaryViewController:`since the split view controller doesn't know after the detail was pushed onto the view controller stack of the master.

In this weird environment that's called iPhone 6(s) Plus, where collapsing and separation of `UISplitViewController`s is possible, only the master's `UINavigationController` knows which soon-to-become detail is on top. [Reaching up](https://sandofsky.com/blog/never-reach-up.html) from a view controller is bad. That's a very strong argument against making any view controller the `UISplitViewControllerDelegate`.

Ah, the joys of legacy code.

The delegate should be something else entirely, it seems. An object which encapsulates so-called knowledge about the way UIKit does its thing. (I should call this "assumptions" instead.)

I cannot come up with something more clever. So here it is:

    #!swift
    import UIKit

    let kViewControllerNoSelection = "NoSelectionNC"

    class CollapsibleSplitViewControllerDelegate: NSObject {

        let storyboard: UIStoryboard
    
        init(storyboard: UIStoryboard) {
        
            self.storyboard = storyboard
        }
    }

    extension CollapsibleSplitViewControllerDelegate: UISplitViewControllerDelegate {
    
        func splitViewController(splitViewController: UISplitViewController, collapseSecondaryViewController secondaryViewController: UIViewController, ontoPrimaryViewController primaryViewController: UIViewController) -> Bool {
        
            if let viewControllerIdentifier = secondaryViewController.restorationIdentifier where viewControllerIdentifier == kViewControllerNoSelection {
            
                // Always collapse empty detail
                return true
            }
        
            // Leave detail on top
            return false
        }
    
        func splitViewController(splitViewController: UISplitViewController, separateSecondaryViewControllerFromPrimaryViewController primaryViewController: UIViewController) -> UIViewController? {
        
            if isDisplayingPasteDetailScene(primaryViewController: primaryViewController) {
                // Keep the detail scene (.None indicates the framework 
                // will figure out what to do)
                return .None
            }
        
            return emptySelectionNavigationController()
        }
    
        func isDisplayingPasteDetailScene(primaryViewController viewController: UIViewController) -> Bool {
        
            guard let navController = viewController as? UINavigationController, detailNavController = navController.topViewController as? UINavigationController else {
            
                return false
            }
        
            guard detailNavController.topViewController is ShiftAssignmentViewController else {
            
                return false
            }
        
            return true
        }
    
        func emptySelectionNavigationController() -> UIViewController {
        
            return storyboard.instantiateViewControllerWithIdentifier(kViewControllerNoSelection)
        }
    }

It fixes my problems, but it doesn't make me like UIKit more. The new stuff helps develop apps quickly and all, but it's a pain to do without producing bad code, apparently.
