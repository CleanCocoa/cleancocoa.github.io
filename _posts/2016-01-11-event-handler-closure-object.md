---
title: How Closures are a Better Event Handler Protocol Alternative
created_at: 2016-01-11 16:22:24 +0100
kind: worklog
tags: [ closure, protocol, oop ]
comments: on
---

I don't like the way I tend to create view controller event handlers. They are almost always just a sink for methods which have nothing in common conceptually but are tied together because of the view's capabilities. So I began experimenting.

## Closures instead of objects

Closures can encapsulate changes. This works well with callbacks for Repositories which fetch entities from your data store. Instead of returning them, you can pass them forward:

    #!swift
    repository.fetchBanana(bananaId) { banana in
        banana.weight = 250
    }

This increases the level of indentation by 1 compared to a synchronous call that returns an optional, but since it's straightforwardly used and there'll be no alternate paths, that's easily bearable. On the upside you gain basic asynchrony: the block will be executed when the repository is ready while the current context can be left immediately.

The best place for this is in some kind of event handler, use case object, or presenter/interactor in VIPER terms. 

The pattern could be used to implement event handlers, too:

    #!swift
    let repository = ...
    let viewController = ...
    viewController.enlargenBananaHandler = { bananaId in
        repository.fetchBanana(bananaId) { 
            $0.weight = 250 }
    }

This way you get rid of classes that implement all the event handlers of a view controller and become tightly coupled to it. In VIPER terms, the Presenter is almost always the view's event handler, too, although in principle you could create two objects for displaying and event handling. When your event handling boils down to closures, you will not need to provide a specialized object at all and can configure the presenter to transform application data to view models.

Setup code like this would go into a [bootstrapper][boots] or similar. The view controller should not create this closure itself and not have knowledge about repositories. This is part of the division of knowledge along the borders of layers. The user interface layer is at the outside, so is persistence. We need something in between to orchestrate their interplay while separating their concerns.

[boots]: /posts/2015/10/bootstrapping-appdelegate/

## Command Factory

Another abstraction which is even weirder is a factory for commands. The factory then knows the repository. Commands are created on the fly. The view will need to understand how the command factory works, obtaining an executable block via `bananaCommands.changeWeight(250)`, for example, and then run it or enqueue it or whatever. The factory takes care of the command's configuration.

This way commands become things to pass around: closures or even objects. The difference to the event handler block parameters from above is that the commands from the factory don't take any arguments. They know everything to be executed already.

## Comparison

Traditionally, I would define a protocol tailored to the view controllers needs as part of the contract to use the view:

    #!swift
    protocol BananaEventHandler {
        func changeWeight(bananaId: BananaID, weight: Int)
    }
    
    class BananaViewController {
        
        var eventHandler: BananaEventHandler?
        var banana: (data: BananaViewModel, bananaId: BananaID)

        func displayBanana(bananaViewModel: BananaViewModel, bananaId: BananaID) {
            banana = (bananaViewModel, bananaId)
        }

        @IBAction func growBanana() {
            let newWeight = banana.data.weight + 50
            eventHandler?.changeWeight(banana.bananaId, newWeight)
        }
    }

With closures as event handlers this becomes more flexible:

    #!swift
    class BananaViewController {
    
        var changeWeightHandler: ((BananaID, Weight) -> Void)?
        // ...

        @IBAction func growBanana() {
            let newWeight = banana.data.weight + 50
            changeWeightHandler?(banana.bananaId, newWeight)
        }
    }

That doesn't improve much at first, but once your view becomes more complex, you may have:

- `changeWeightEventHandler`
- `throwAwayEventHandler`
- `orderSimilarEventHandler`

The actual implementations of these three can be independent from one another. With a single protocol, the actual event handler would have to delegate to three different use case object -- because the actions don't have anything in common, it doesn't make sense to implement the logic of all three actions in a single object.

With closures, the glue is not an event handler object anymore. I can pass the specialized use cases' handlers directly:

    #!swift
    viewController.changeWeightEventHandler = resizeBanana.changeWeight
    viewController.throwAwayEventHandler = disposer.disposeBanana
    viewController.orderSimilarEventHandler = orderDispatcher.orderSimilar

Instead of creating closures, I assign method pointers. Works the same way.

So I end up with three service objects that do their specialized job well and provide an interface to be compatible with the view controller.

The downside: this coupling is only visible in the above setup code. There's no dependency via type which the protocol approach made transparent.
