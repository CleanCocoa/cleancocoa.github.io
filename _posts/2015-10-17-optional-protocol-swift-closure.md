---
title: Optional Protocol Methods in Swift using Closures and No Protocol, Actually
created_at: 2015-10-17 14:09:39 +0200
kind: worklog
tags: [ protocol, swift ]
vgwort: http://vg08.met.vgwort.de/na/5951969d91774af2b8aa68a5680e9a5e
comments: on
---


So you miss declaring optional methods in Swift protocols? [Joe Conway](http://blog.stablekernel.com/optional-protocol-methods-in-pure-swift)
got you covered: "optional methods can be implemented by providing a default implementation in an extension." But there's an even better way, in my opinion: ditch the notion of optional methods and use blocks instead.

Joe's clever way to bring back optional methods is to use protocol extensions. When protocol extensions provide an initializer already, the type implementing the protocol doesn't have to:

    #!swift
    protocol ExampleDelegate {
        func numericValueForObject(object: DelegatingObject) -> Int
        func objectShouldDoMoreWork(object: DelegatingObject) -> Bool
    }
    ​
    extension ExampleDelegate {
        func numericValueForObject(object: DelegatingObject) -> Int {
            return 0
        }
        func objectShouldDoMoreWork(object: DelegatingObject) -> Bool {
            return false
        }    
    }
    
    class ConcreteDelegate: ExampleDelegate {
        // ... no need to provide the protocol methods here.
    }

Then the caller, which is probably a UI-component of some kind, can implement the defaults itself to not duplicate code in the event that no delegate was set:

    #!swift
    class DelegatingViewController {
    
        var delegate: ExampleDelegate?
      
        // ...
      
        func stuff() {
            // Query the delegate or, if none is set, use protocol defaults
            // this object itself inherits in a second.
            var numericValue = (delegate ?? self).numericValueForObject(self)
            ...
        }
    }
    
    // Make the call to `self.numericValueForObject` work:
    extension DelegatingObject : ExampleDelegate {}

I don't like to [use mixins or traits][mixin] like that because they add responsibilities to the type.

[mixin]: /posts/2015/09/swift-protocol-as-trait-testing/

Taking the reasoning a step further: optional protocol methods were used by Cocoa and UIKit a lot. But why should _you_ create an API that requires optional methods as part of the protocols?

* If it's a "data source"-like method, it probably should be required anyway.
* If it's a callback, use closures instead of protocols.

Chances are you'd like to design a delegate protocol to react to all events the view controller can emit. In the case of `BananasViewController` and its `BananasEventHandlerProtocol`, the events may be: `peel(banana)` and `throwAway(banana)`. 

Now if not all concrete handlers absolutely _have_ to supply both methods, what can you do instead of using Joe's technique?

Instead of adding these to a protocol which is always tightly coupled to the view anyway, add single event handlers to the view controller:

    #!swift
    class BananasViewController {
    
      var peelHandler: ((Banana) -> Void)?
      var throwAwayHandler: ((Banana) -> Void)?
      
      // ...
      
      @IBAction peel(sender: AnyObject) {
      
        let banana = currentlySelectedBanana()
        
        peelHandler?(banana)
      }
    }

Then you can wire any object with any method satisfying the signature requirements to the view controller.

    #!swift
    class BananaPeeler {
        func peelTheBanana(banana: Banana) {
            //...
        }
    }
    
    class RemoveBanana {
        func trashYellowFruit(banana: Banana) {
            // ...
        }
    }
    
    // somewhere in your set-up code
    let peeler = BananaPeeler()
    let remover = RemoveBanana()
    bananaViewController.peelHandler = peeler.peelTheBanana
    bananaViewController.throwAwayHandler = remover.trashYellowFruit

Of course this can go wrong if you wire `RemoveBanana`'s method to the `peelHandler` property by accident: both have the same signature, so the compiler won't complain. But unit tests will rescue you.

Note that I picked weird method names without affecting the view controller. Thanks to blocks and methods being first-class objects, it doesn't matter how the actual handler is called. That's another notable difference to using protocols.

I'm experimenting with this myself at the moment and am quite happy. Instead of a generic and all-purpose `FileEventHandler` which implemented a protocol to add, remove, and group `File` objects, I now have super-focused use-case driven objects like `AddFile` and `RemoveFile`. 

`AddFile` contains a method with the proper signature and is wired to the view controller [during bootstrapping][bootstrap].

Writing tests is easier, too. The all-purpose `FileEventHandler` was supposed to interact with a variety of other objects. Since there were too many to make coding comfortable, I was hesitant to add new ones. Nobody likes to inject a lot of collaborating objects through an initializer. More than 4 make me nervous.

So in the end I was relived to have split the beast into multiple parts. Not having to define a ton of different protocols to achieve that helped, too.

It's a pretty safe refactoring to transition from a protocol to optional closure attributes: the past protocol implementations will just work if you keep the signatures.

[bootstrap]: /posts/2015/10/bootstrapping-appdelegate/
