---
title: Separate Read Model from Write Model to Support Complex Forms
created_at: 2015-12-19 15:04:03 +0100
kind: worklog
tags: [ mvvm, uitableview, form, cqrs ]
vgwort: http://vg01.met.vgwort.de/na/ab01dab9ae4447acaadd29929fb91f96
comments: on
---

One thing that repeatedly messes up the conceptual purity of a view model is figuring out which entity should be mutated upon user interaction. Part of this purity stems from the fact that a view model is [best served as view data][data] to stress that it doesn't contain much (business) logic. Making the data mutable introduces a lot of problems figuring out what that view model really _is._

[data]: http://khanlou.com/2015/12/mvvm-is-not-very-good/

In [a post on MVVM and presenters][mvvm] I illustrated that the view model is tailored to the view's needs. But like every example mine, too, is focused but contrived, and so the code fails to show an important feature: telling any kind of controller or service which entity should be updated or deleted when the user presses the appropriate buttons.

[mvvm]: /posts/2015/10/view-model-control/

The view model ideally should only contain display data in a format that is easily consumed by a view component. (Most likely strings and images.) But to associate the view model to the underlying entity, you'll need to add an attribute the view doesn't care about: the entity's identifier.

## Conceptual purity

Once you add the identifier, the view model is not exclusively coupled closely to the view anymore -- it also fulfills needs of the adjacent application layer which interacts with the database, for example. The coupling is only implicit since the view model doesn't directly talk to any other service object. It's supposed to be mostly data representation.

Let's assume that conceptual purity is worth the effort. (I'm not saying it is.) How can we separate entity identification from view model representation?

One way is to use tuples to associate two values. A tuple like `(BananaID, BananaViewModel)` is the little brother of a struct with just two attributes. To use tuples feels like duck-taping and bad style to me, but I think this is just another presupposition I have to get rid of to advance in Swift.

A tuple is at least as expressive as a struct without behavior if you name its parts:

    #!swift
    let values: (bananaId: BananaId, bananaData: BananaData) 
        = (BananaId(123), BananaData(weight: 200))
    print(values.bananaData.weight) // => 200

Give the tuple a `typealias` and it'll be hard to distinguish them from structs in client code.

But what name would this be? What name fits the combination of ID and view model data?

## Split read from write model

Thinking about my present problem domain, I think the underlying problem is that the view model isn't pure. It's currently mutable to react to user changes. But that's not its responsibility.

Instead, there should be:

- View model to display data
- Entity ID passed in, too, for later association of the result
- Command model with instructions to change the entity
- Combination of entity ID and command model

(Welcome to [CQRS][], by the way.)

I've written about a command model [before][command]. It can be `NewBanana`, containing data to construct a new entity. Or it can be some kind of abstraction of the underlying change.

Placing orders makes this even more clear:

- `OfferDetails` contains the item's data for display
- `NewOrder` holds quantity, for example (I like the `NewXYZ` convention a lot)

This relieves the view model from a lot of burden. But the ID is still free-floating, not associated with the view model at all.

[cqrs]: http://martinfowler.com/bliki/CQRS.html

## Where does ID belong? Read or write?

If the order and checkout process spans across 3 view controller scenes, for example, I'm satisfied to populate `NewOrder` over time and pass the immutable `OfferDetails` around with it with every hop. 

But what's the ID part of? Offer, because they share an origin as *read* data, or Order, because it's going to be *written* to the database?

It makes sense to separate both value types. And it works well to pass them around using a `showOrder(_:, forOffer:)` method. This message is sufficiently cohesive as it is. But taking it a step further, I ponder grouping both in a `Transaction` type to make sure none of the halves get lost or replaced in the process.

My main motivator for that refactoring is that both objects will be passed around for quite some time. If it belongs together semantically, put it together.

The possible downside of this is that the transaction will have to act as a facade and expose an interface that under the hood delegates to the read or write model.

So here's my concluding hunch so far, ordered by importance:

- Separate read model from write model
- Associate the ID with the write model -- because it'll be used during the write operation
- Group both models in some kind of transactional type. Maybe use `PlaceOrder` as facade (the internal read/write model names suddenly don't matter to the client anymore) and offer a factory to create `NewOrder` data structures which are used for persistence

When I began thinking this through for the particular problem at hand, I was really annoyed by introducing a new type as the write model and then refactoring view display to use that.

But the clarity and focus of components improved. They became slimmer and more readable. At least a bit. Tests looked good, too, and creating value objects with less attributes was nice,

Even nicer was that different views could now adopt a `DisplaysOrder` protocol without implying to perform changes -- because they just can't change what's immutable. 
