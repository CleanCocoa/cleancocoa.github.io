---
title: 3 Ways to Model Relationships in a Domain
created_at: 2015-03-11 17:04:28 +0100
kind: worklog
tags: [ domain-model, aggregate-root, service ]
vgwort: http://vg08.met.vgwort.de/na/6e9a2fb06f304734afc322c7cfdabf23
comments: on
code: swift
---



Let's assume we need a [Domain Model][domain] and can't create our application with basic [CRUD][] actions and data containers.  Let's further assume we have a `Box` Entity which can contain many `Item`n. `Item` is an Entity, too, because its attributes may change over time, so a Value Object won't do.

You may replace these two with almost any one-to-many relationship parts. How do we model this? Which criteria help make decisions during the modeling process?

* Naive data-centric approaches use collection objects to hold references directly.
* Even if data containers become Aggregate Roots of your Domain will you like to keep the collections behind the scenes.
* If transactional consistency is at stake for complex operations, you'd better tear both Entities apart to keep the data safe.

## Data-Centric

The obvious solution is to make it so that `Box` contains `Item`s:

```swift
class Item {
    let title: String
}
class Box {
    var items = [Item]()
}
```

That's it for managing the data relationship. Client code has full access to the `items` array: it adds, removes, reorders, and maybe even breaks it directly.

To allow only certain operations, we'd have to hide the collection. Assume this is the case. In terms of Domain-Driven Design, we then have not reached the end, yet.

## Introducing Aggregates and Their Roots

In this case, a `Box` is the Aggregate Root. The Aggregate consists of a `Box` with many `Item`s. 

Consequently, if we access `Item`s, we can only do so through a `Box`. We want to add, remove, and iterate over them. Since `Item`s are not meant to be in any particular order, it doesn't make sense to query them by index.

```swift
class Box {
    private var items = [Item]()
    
    func addItem(item: Item) { ... }
    func removeItem(item: Item) { ... }
    func eachItem(block: (Item) -> ()) { ... }
}
```

All interactions flow through `Box`. `Item`s are essentially and conceptually a part of `Box`. The collection behind the scenes is hidden from client code. If something wants to modify the contents of a `Box`, it has to do so through its public interface. 

An even stricter version may hide `Item` creation altogether and only allow indirect modifications. `Item`s are created and referenced by their title attribute, which may suffice in some cases and totally depends on the specifications:

```swift
class StrictBox {
    private var items = [Item]()
    
    func addItem(#itemTitle: String) { ... }
    func removeItem(#itemTitle: String) { ... }
    func eachItemTitle(block: (String) -> ()) { ... }
}
```

That's it for the proof of concept. Let us stick to the less strict version.

Say we want to move `Item`s around between `Box`es. `Item`s, being Entities themselves, have an identity of their own over time. To delete one instance and add another with similar attributes instead of moving the real thing around doesn't fit the language we use. Keeping the model close to our language, we may come up with this:

```swift
extension Box {
    func moveItem(item: Item, toBox: Box) {
        removeItem(item)
        toBox.addItem(item)
    }
}
```

I chose to add the `items: [Item]` attribute to `Box` early on because this is my naive take on one-to-many relationship resolution. Some persistence mechanisms may actually benefit from this. (Core Data managed objects for example.) Some begin to choke because you have to keep your convenient collection in sync with changing data in the data store.

## Separating Aggregate Roots

Instead of modeling the relationship from `Box` to `Item` as one of containment, couldn't both be created equal?

I argue it makes sense to make both Entities their own Aggregate's Root in some cases. It does so right away when the business rules state that `Item`s may also exist outside of `Box`es for a while.

As a rule of thumb, Aggregate Roots should reference each other via identity. These are Value Objects I like to create as `struct`s in Swift. So assume we have both `BoxId` and `ItemId` which hold a 64-bit Integer or a string UUID or what have you. (In fact, for the sake of persistence, I would use these anyway, but only now do they become important for the example.)

Now we have to change things a bit and replace the class definitions with the following:

```swift
class Item {
    var boxId: BoxId?
    let itemId: ItemId
    let title: String
    
    var isLyingAround: Bool {
        return boxId == nil
    }
}

class Box {
    let boxId: BoxId
}
```

Since `Item` is responsible for itself, moving it around can be modeled thus:

```swift
extension Item {
    func moveToBox(box: Box) {
        boxId = box.boxId
    }
    
    func layOnFloor() {
        boxId = nil
    }
}
```

Say we need to print a list of a `Box`'s contents. At least at this point do we need a means to find all `Item`s of a given `Box`.

Since `Box` doesn't own the `Item` objects anymore, we need a Service Object in the Domain (!) to hold the business logic. Also, I add the Repository protocol definition to accessing the data store at this point so you know how things work:

```swift
protocol ItemRepository {
    func items(#boxId: BoxId) -> [Item]
    func nextId() -> ItemId
    func addItem(item: Item)
    func removeItem(itemId: ItemId)
}

class PrintBoxContents {
    let itemRepository: ItemRepository
    
    func print(box: Box, printer: Printer) {
        let boxId = box.boxId
        let items = itemRepository.items(boxId: boxId)
        
        printer.printContentList(box, items)
    }
}
```

To find all `Item`s with a given `BoxId` is now the adequate way to de-reference associations.

Assume we should limit creating new `Item`s to the scope of a `Box`, and the lying-on-the-floor case should not be encouraged by creating free-floating `Item`s, we have to do something like this:

```swift
extension Box {
    func item(title: String, identityProvider: ItemIdentityProvider) {
        let itemId = identityProvider.nextId()
        return Item(itemId: itemId, title: title, boxId: self.boxId)
    }
}

extension Item {
    init(itemId: ItemId, title: String, boxId: BoxId) { ... }
}

class ItemIdentityProvider {
    let itemRepository: ItemRepository
    
    func nextId() -> ItemId {
        return itemRepository.nextId()
    }
}
```

Why this weird `ProvidesItemIdentity` dance? Because injecting the `ItemRepository` into `Box.item(...)` would expose too much power to `Box`.

The simple `ItemIdentityProvider` is a seemingly bloated Domain Service. But it's there to keep the data safe at all times. It's there to shield the Repository from access from unexpected places. This is the same rationale behind making the collection `item: [Item]` private at the beginning.

Still, `Box` doesn't know how many `Item`s it contains. Domain Services have to query the Repository and find out to provide the numbers. Only via Repository can this knowledge be obtained, and `Box` probably shouldn't have access to it if we can help it.

## Taste VS Rationale

I liked the first version better: I liked it when it was possible to call `box.addItem(item)`, because it expresses the intent so clearly. Now, with the Domain Services doing all the interesting things, the Domain Model became quite [anemic][].

Some credit is due to this overly simple and contrived example. In real-world use cases, you'd have more Entities and more behavior to attach to them.

If your Entities are nothing more than dumb containers for other Entities, you probably fare better without a Domain Model in the first place. Simply go for [CRUD][], maybe keep the Repositories for this, and there you go.

Why would you want to model two Entities as separate Aggregate Roots?

The consistency boundary of transactions plays a big role in deciding this. If you want to remove a `Box` and re-distribute its contents among other `Box`es evenly, you modify at least two Aggregate's contents in each transactions: source and destination `Box`.

If `Item` is its own Aggregate Root, though, you modify only one Aggregate per transaction, though emptying a `Box` fully consists of multiple transactions in sequence. That's a strong argument for tearing them apart, even if the models themselves become a bit less expressive. But then again, your model objects will probably be not as simple as the ones I presented.

Reason about your Entities and their transactions to find out where to draw the line. Leave business logic in the Domain as Services nevertheless. It'll still take you far, even though you will end up with more classes than before.

[domain]: http://martinfowler.com/eaaCatalog/domainModel.html
[crud]: http://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[anemic]: http://www.martinfowler.com/bliki/AnemicDomainModel.html
