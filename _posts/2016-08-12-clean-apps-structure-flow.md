---
title: When You Code, You Design Both Structure and Information Flow
created_at: 2016-08-12 08:05:54 +0200
kind: worklog
tags: [ clean-code, 1df, software-architecture, reswift, viper ]
vgwort: http://vg01.met.vgwort.de/na/494e81218ce7437faf4e8eb428a242ab
comments: on
---

As soon as you write a piece of software, you "architect" it. Can't get around that; but if you do not do it consciously, the resulting structure may not be great. Taking ownership of the process is important to change the result and create maintainable software.

When we write/architect software, we worry about two things:

1. **Data flow**, a.k.a. "process". This manifests during runtime, when messages are sent and variables changed.
2. **Structure** of components in code, module design, project setup, including file names and folders. I wrote about this part in length [earlier this week](/posts/2016/08/mvvm-is-okay-for-what-it-does/).

When you create two classes and delegate an action from A to B, you create a structure _and_ design the data flow. The two are intertwined and bringing them into existence essentially equals "doing software architecture" (for our purpose at least).

Why split this up analytically, then?

We should consider these two parts of architecture distinct because it adds tremendous value: you can change the flow without replacing all of the structure. Since flow and structure are intertwined, changing one will affect the other. But if the static structure seems all okay, you now can decide to keep it mostly unchanged and focus on the flow of information instead. You just have to be able to separate these two things.

This isn't hard at the beginning. But the cognitive load adds up as the code base grows and time passes. You have to re-read code again and infer both structure and flow of the component you're looking at: when is this method called? Where does the result go? Which other object is involved in the process? How tightly are they coupled, and can I re-use them?

This is hard work, mentally. Bad code makes the work even harder.

So I came up with this heuristic: **Clean code reveals structure and processes easily.**

When I set out to write clean code, improving the structure is only one part of the progress. I can split up a huge view controller into 10 objects but end up passing messages around erratically, making it _harder_ to find out what happens when the user presses a button, say.

----

If flow and structure are somewhat independent, you may understand why using VIPER as the basic architectural pattern _and_ using ReSwift to handle information flow and model updates works.

VIPER is about the structure. If you create the Wireframe which sets up a Presenter to an Interactor and a View component, you end up with a dependency graph of objects. That's the structure I'm talking about.

ReSwift worries about the flow. It requires a Store and an app State object. Updates to the State are handled by the Store's Reducers; the result is then passed to all interested parties. Here, you model information flow: user-triggered event objects affect the state; the state is reported back to the rest of the app. ReSwift introduces a front-end/back-end distinction, so to say, as part of its structural requirements.

Combining the two, you'll find out that the "pull" mindset of a VIPER Interactor is outdated: user actions are handled by the data flow thanks to ReSwift. The Interactor doesn't need to fetch stuff from a database anymore; changes to the `visibleStuff` collection in the State are pushed to observers. Now who's the observer? The Presenter could be the observer, handling ReSwift's callback `newState(_:)` and call its `updateView(_:)` method after that. The Interactor of this particular VIPER component is obsolete.

If the transformation from app State to data for the view is complex, maybe the Presenter isn't the best place for that. In that case, I'd leave the Interactor in place. But instead of pulling data inside the Interactor, the Interactor becomes the receiver of ReSwift's pushy notification. It can transform the data to a format suitable for this VIPER component and then pass it to its "output port," that is the callback you have wired the Presenter to.

Like so, in simplified code:

    #!swift
    class BananaInteractor {
        var outputPort: ((BananaViewModel) -> Void)?
    }
    
    extension BananaInteractor: ReSwift.StoreSubscriber {
        func newState(appState: AppState) {
            let bananaState = ... // from appStare
            outputPort?(bananaState)
        }
    }
    
    protocol BananaView { 
        func displayBanana(bananaViewModel: BananaViewModel)
    }
    
    class BananaPresenter {
        let view: BananaView
        
        func presentBanana(bananaViewModel: BananaViewModel) {
            view.displayBanana(bananaViewModel)
        }
    }
    
    class BananaViewController: UIViewController, BananaView { ... }
    
    class BananaWireframe {
        // ...
        func setup() {
            view = BananaViewController()

            presenter = BananaPresenter(view: self.view)

            interactor = BananaInteractor()
            interactor.outputPort = { [weak self] in 
                self?.presenter.presentBanana($0) 
            }
        }
    }
