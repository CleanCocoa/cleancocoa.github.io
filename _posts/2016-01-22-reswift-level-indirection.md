---
title: ReSwift, Law of Demeter, and Another Level of Indirection
created_at: 2016-01-22 10:21:24 +0100
kind: worklog
tags: [ reswift, lod, state, flow, oop, 1df ]
shared_image: "blog/201601212226_reswift_concept.png"
vgwort: http://vg01.met.vgwort.de/na/a930cfd49784492e8e20c29a5681cb34
comments: on
---


Benjamin Encz's presentation ["Unidirectional Data Flow in Swift"][en] about [ReSwift][re] features global app state: there's one `AppState` type that acts as the facade to model and navigation state which is the single point of truth of every state in the app. This is a game changer when you suffer from massive view controller syndrome. In this post, I'd like to show you how he envisions the state of an app and what a next step could look like.

## Global AppState

Here's my version of the `AppState` that includes Benjamin's recommendations presented in the Q&A after the presentation:

    #!swift
    protocol KnowsTheCount {
        var counter: Int { get }
    }
    
    struct AppState: StateType, HasNavigationState, KnowsTheCount {
        
        // Model state, e.g.:
        var counter = 0
        
        // Navigation state
        var navigationState = NavigationState()
    }

The most basic type of app state that includes routing to view controllers is the combined `protocol<StateType, HasNavigationState>`. That's a combination of what the router on one side and the main state store and its reducers expect. 

{% include figure.md src="/assets/blog/201601212226_reswift_concept.png" alt="data flow" caption="Flow of actions to change state and propagate the change back to the app." %}

I've written a few words about [reducers and actions](/posts/2016/01/event-declarative-state/) yesterday already. It's a really cool concept on its own already.

Consumers of the state like views can register for changes and obtain the current app state. When you add the model state specialization `KnowsTheCount` like I did, you can make the consumer obtain not just an instance of `AppState` with all its knowledge about everything that's going on, but an instance of `KnowsTheCount` specifically. This enables view components to display the model value without potentially probing navigation state for whatever reason. On the consumer's side, _restricting access makes clear what_ is needed -- in this case, the current model value, nothing else. When you come back to the code and want to change something, you don't have to think about all the other properties that the underlying `AppState` exposes. You'll just see what you're interested in.


## Ditch the habit of reaching deep into your objects

There's a little caveat, though: consumers will obtain a model object of type `KnowsTheCount` and request the `counter` value this protocol publishes. That's reaching two levels deep. If you care about the [Law of Demeter](https://en.wikipedia.org/wiki/Law_of_Demeter) (which is only a weak principle, mind you), you will want to restrict access only to "close friends". 

Here's an example:

    #!swift
    // Not passing the strict LoD test:
    func displayBananaSize(banana: Banana) {
        self.sizeLabel.text = "\(banana.size)"
    }
    
    // Passing the strict test:
    func displayBananaSize(size: Size) {
        self.sizeLabel.text = "\(size)"
    }

Now the Law of Demeter has no worth in itself. It's just a gentle reminder not to reach too deep into other objects or you will increase implicit coupling _a lot._ Asking a collaborator for one of its attributes is not the worst thing that can happen. 

But, you know, when the deadline's approaching, maybe reaching 2 or 3 levels deep won't hurt you a lot right now, would it? -- And thus code degrades.

To prevent this from happening, let me show you how applying the oddly named Law of Demeter can help you build a habit to create more solid public interfaces.

One of the simpler cures against reaching for an object's attributes is inverting the flow of information. I adopted the term "east-oriented" as a paradigm of thinking in terms of passing info along with commands instead of querying for it. Instead of designing components that request values from model objects, design model objects that actively display their state in components.

The banana example can be enhanced like this:

    #!swift
    // East-oriented variant
    protocol SizeComponent {
        func displayBananaSize(size: Size)
    }
    
    extension Banana {
        func displaySize(component: SizeComponent) {
            // component is a friend we may trust 
            // self.size is a very well known & trustworthy friend
            component.displayBananaSize(self.size)
        }
    }
  
    extension ViewComponent: SizeComponent {
        // Even another level of indirection!!!11
        func displaySizeOfBanana(banana: Banana) {
            banana.displaySize(self)
        }

        func displayBananaSize(size: Size) {
            self.sizeLabel.text = "\(size)"
        }
    }

You see that a `Banana` takes care of displaying its size in an appropriate component. Instead of creating explicit protocols, which can get unnerving quickly, you could model the method with a callback like `displaySize(component: (Size) -> Void)`.

## Applying the LoD to ReSwift

Now back to ReSwift: the `AppState` as I showed it already partitions itself using protocols and, when the complexity grows acceptably, delegates to sub-state components for details.

Next we could make the app state expose _behavior_, so that view components can request display of some aspect in themselves just like the `ViewCompontent` in the example above did.

The following example assumes there's a `mainStore` accessible somehow where observers can register for state changes. Observers have to implement the [`StoreSubscriber`](https://github.com/ReSwift/ReSwift/blob/master/ReSwift/CoreTypes/StoreSubscriber.swift) protocol as shown below:

    #!swift
    protocol CountComponent {
        func newCount(count: Int)
    }
    
    protocol KnowsTheCount {
        var counter: Int { get }
        func provideCount(comp: CountComponent)
    }
    
    struct AppState: StateType, HasNavigationState {
        
        // Model state, e.g.:
        var counter = 0
        
        // Navigation state
        var navigationState = NavigationState()
    }
    
    extension AppState: KnowsTheCount {
        func provideCount(comp: CountComponent) {
            comp.newCount(counter)
        }
    }
    
    // for the protocol, see:
    //   <https://github.com/ReSwift/ReSwift/blob/master/ReSwift/CoreTypes/StoreSubscriber.swift>
    class SomeViewController: UIViewController, StoreSubscriber {
        
        typealias StoreSubscriberStateType = KnowsTheCount
        
        override func viewWillAppear() {
            mainStore.subscribe(self)
        }
        
        override func viewWillDisappear() {
            mainStore.unsubscribe(self)
        }
        
        // The store's `StoreSubscriber` state change callback
        func newState(state: KnowsTheCount) {
            state.provideCount(self)
        }
    }
    
    extension SomeViewController: CountComponent {
    
        func newCount(count: Int) {
            self.veryBigCountLabel.text = "\(count)"
        }
    }

The wording is a bit clumsy (`provideCount`?!), but with a real application you'll easily come up with better names.

Bear in mind I haven't tested the original approach nor my changes in a real-world shipping application -- I just _imagine_ this additional level of indirection to help hide information from the global `AppState` facade from consumers and make things actually simpler.

Happy to hear ideas and critique!

[re]: http://reswift.github.io/ReSwift/master/
[en]: https://realm.io/news/benji-encz-unidirectional-data-flow-swift/
