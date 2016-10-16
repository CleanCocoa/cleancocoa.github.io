---
title: Type-Safe Storyboard Segues
created_at: 2016-04-25 09:46:33 +0200
kind: worklog
tags: [ segue, enum, protocol, mixin ]
vgwort: http://vg01.met.vgwort.de/na/03d743ccb4e94b389c5c9a08a80ec00f
comments: on
---

I thought I had [my](/posts/2015/12/wwdc-unwind-segues/) [share](/posts/2015/01/segues-vs-tell-dont-ask/) of Storyboard Segue-related tips already to make them less brittle when I found yet [another approach to use Swift's enums for segues][post] by Ricardo Pereira.

His team uses enums not for identifiers but to associate model data with the segues:

    #!swift
	enum StoryboardDestination {  
	    case Login
	    case DeviceGroups(userAuth: UserAuthentication)
	    case Devices(userAuth: UserAuthentication, model: DeviceGroup)
	}

These have to be translated to Strings in order to programmatically perform a segue. You can only either have associated values in enum cases *or* [make a segue identifier enum inherit from String](/posts/2015/06/segue-enums/), but not both, sadly. Somewhere you'll have to place manual translation to String. For their app Qold, they did it in an extension:

    #!swift
    extension UIViewController {
    
        func performSegueWithIdentifier(destination: StoryboardDestination) {
            let segue: String
    
            switch destination {
            case .Login:
                segue = "LoginSegue"
            case .DeviceGroups(_):
                segue = "DeviceGroupsSegue"
            case .Devices(_, _):
                segue = "DeviceGroupDetailsSegue"
            }
    
            performSegueWithIdentifier(segue, sender: Box(destination))
        }
    }

The method name should've simple been `performSegue(destination:)`, but hey.

The enum value gets boxed and then becomes the `sender` of that command. In `prepareForSegue`, you can unbox it and use the associated values as model data for the destination view controller. 

With segues in general, I never liked that the source view controller is supposed to configure the destination view controller. The destination view controller thus gets pushed around. Maybe not much, but it's not my favorite thing to do. I like to be polite to objects.

Ricardo got me hooked when he shows how they handle the configuration: the destination view controller has to conform to `StoryboardDependencies` to configure itself in `assignDependencies`:

    #!swift
    class StoryboardViewController: UIViewController {
    
        override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    
            // ...
            if let destinationViewController = anyDestination as? StoryboardDependencies,
                let dependencies = sender as? Box<StoryboardDestination> {
                    destinationViewController.assignDependencies(dependencies)
            }
        }
    }

Even "configure" would've been a better choice for a method name than "assign dependencies" from the destination view controller's point of view (which is the one that counts). The destination view controller actually _takes_ (not assigns) the dependencies:

    #!swift
    class DevicesViewController: StoryboardViewController, StoryboardDependencies {
    
        func assignDependencies(dependencies: Box<StoryboardDestination>) {
            switch dependencies.value {
            case .Devices(let userAuth, let model):
                self.userAuth = userAuth
                deviceGroup = DeviceGroupViewModel(model)
            default:
                break
            }
        }
    }

Otherwise, this approach looks solid to me.

I don't know why the boxed value is transmitted, though; having to use a `Box` type in some cases in unfortunate but necessary. Nevertheless this is a code smell. I wouldn't want to propagate this too far in my app and unbox in `StoryboardViewController.prepareForSegue(_:)` already. All of these are minor issues and can be remedied easily in my own code.

## Making the convenience method work in other view controller subclasses

`StoryboardViewController` encapsulates supporting different kinds of segue configurations. This is great because if you stick to using the `StoryboardDestination` enum, everything in terms of segue setup now comes for free.

But only if you have view controllers subclass `StoryboardViewController`. If you inherit from a `UITableViewController`, you're out of luck at first.

Thanks to protocol extensions, we can remedy this at least a bit.

    #!swift
    protocol PreparesSegues {
        func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?)
    }
    
    extension PreparesSegues {
        func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
            // code from StoryboardViewController
        }
    }

This is a typical example of using protocols as mixins: you can remove `StoryboardViewController` and extend any view controller subclass with this protocol to get the convenient version of `prepareForSegue(_:, sender:)` for free again.

[post]: https://www.whitesmith.co/blog/safe-segues-in-storyboards/
