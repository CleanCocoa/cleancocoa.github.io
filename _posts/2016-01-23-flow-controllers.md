---
title: Flow Controllers to Replace Segue-based View Transitions
created_at: 2016-01-23 12:09:24 +0100
kind: worklog
tags: [ state, change, segue, navigation ]
url: http://merowing.info/2016/01/improve-your-ios-architecture-with-flowcontrollers/
comments: on
preview: fulltext
---

[Krzysztof Zabłocki](http://merowing.info/2016/01/improve-your-ios-architecture-with-flowcontrollers/) wrote about the concept of "Flow Controllers". Assimilating the concept, I imagine they're similar to the [bootstrappers](/posts/2015/10/bootstrapping-appdelegate/) I use during app launch, only Krzysztof says there's just one initial flow controller after launch while the others are used when transitions need to take place.

Separating [state from transitions](/posts/2016/01/separating-state-change/) is a higher-level goal of architecting apps -- and is especially amiss in `UIViewController`-centered iOS apps. "Flow Controllers" encapsulate the change or transition. This includes setting up the view controller according to the presentation needs: 

> * Configuring view controller for specific context - e.g. different configuration for an Image Picker shown from application Create Post screen and different when changing user avatar
> * Listening to important events on each ViewController and using that to coordinate flow between them.
> * Providing view controller with objects it needs to fulfill it’s role, thus removing any need for singletons in VC’s

The singleton Krzysztof talks about is the usual way app state is modelled.

Here's a slightly modified example to illustrate how a flow controller manages transitions between view controller scenes with `configureProgramsViewController`, `configureCreateProgramViewController`, or similar:

    #!swift
    func configureProgramsViewController(viewController: ProgramsViewController, 
        navigationController: UINavigationController) {
        
        viewController.state = state
        
        // Add button action callback to show the next scene
        viewController.addProgram = { [weak self] _ in
            guard let strongSelf = self else { return }
            
            let createVC = // recreate a view controller, e.g. from a Storyboard
            strongSelf.configureCreateProgramViewController(createVC, 
                navigationController: navigationController)
            navigationController.pushViewController(createVC, animated: true)
        }
    }

The flow controller owns the state here and takes care of pushing view controllers onto the navigation stack. This will not work with segues. It's a replacement for them.
