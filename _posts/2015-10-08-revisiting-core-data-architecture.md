---
title: Revisiting the Core Data + Domain Model Architecture
created_at: 2015-10-08 08:40:20 +0200
kind: worklog
tags: [ core-data, ddd, model, domain, software-architecture, smell ]
comments: on
---

It happens that just yesterday I read about architecture smells in code. Among the examples was "subclasses don't redefine methods". In [my post about Core Data and expressive domains][doms] earlier this week, I did just that: create a `Egg` subclass of `CoreDataEgg` to inherit `CoreDataEgg`'s behavior. That's not what abstraction to superclasses is meant to do.

So I cheated in the post's example. That made me wonder if I should move logic in my application down to `CoreDataXYZ`, and if so, which.

Imagine my app not only deals with `Egg`s, but also with their `Basket`s. An egg can be put into a basket. It can also be moved between baskets.

Since eggs can do so much more, it's necessary to model an expressive domain. It doesn't suffice to have [CRUD][] actions and use Core Data to conveniently access the database. If that were the case, I would use an `EggStore` service object which fetches managed objects from the Core Data stack and transforms them into value objects, aka `structs`. But that's sadly not the case.

Imagine there's a `Basket` and its `CoreDataBasket` superclass. A basket has a unique `BasketID` which is used for events throughout the app.

So we end up with this:

    #!swift
    @objc(Egg)
    class Egg: CoreDataEgg {
        
        // This is new
        
        func moveToBasket(basket: Basket) {
                        
            removeFromCurrentBasket()
            
            currentBasket = basket
            
            DomainEventPublisher.sharedInstance
                .publish(EggMovedToBasket(eggID: eggID, basket: basket.basketID))
        }
        
        private func removeFromCurrentBasket() {
            
            guard let oldBasketID = basketID else {
                return
            }
            
            currentBasket = .None
            
            DomainEventPublisher.sharedInstance
                .publish(EggRemovedFromBasket(eggID: eggID, basket: oldBasketID))
        }
        
        var isInBasket: Bool {
            return hasValue(basketID) 
            // via <http://owensd.io/2015/05/12/optionals-if-let.html>
        }
        
        var basketID: BasketID? {
            return currentBasket.basketID
        }
        
        
        // This is from before
        
        var eggID: EggID {
            return EggID(identifier: eggIdentifier)
        }
        
        var size: EggSize {
            return EggSize(rawValue: eggSize)
        }
        
        func changeSize(newSize: EggSize) {
            
            let oldSize = size
            eggSize = newSize.rawValue
            
            DomainEventPublisher.sharedInstance
                .publish(EggSizeChanged(eggID: eggID, from: oldSize, to: newSize))
        }
    }
    
    // Living in the same file to make use of per-file private properties
    class CoreDataEgg: NSManagedObject {
        
        static let entityName = "Egg"
        
        static func fetchRequest() -> CoreDataFetchRequest<Group> {
    
            return CoreDataFetchRequest(entityType: Group.self)
        }

        static func insert(newEgg: NewEgg, context: NSManagedObjectContext) {
    
            let egg = NSEntityDescription.insertNewObjectForEntityForName(entityName, inManagedObjectContext: context) as! Egg
    
            egg.eggSize = newEgg.size.rawValue
            egg.eggIdentifier = newEgg.eggID
        }
        
        @NSManaged private var eggIdentifier: String
        @NSManaged private var eggSize: Int
        @NSManaged private var basket: Basket
    }

I've added the basket-related logic to `Egg`, as you can see.

Subclassing a managed object normally poses the problem that you could set the `basket` property directly. Thanks to it being private, that's not easily possible. I can still work around this with Key-Value Coding, but then I'd be an idiot for screwing with my own code conventions. No one wants to be called an idiot, so I don't do that.

The `Egg` Domain Model object takes care of event delivery. That's all there is in terms of business logic at this point: change the basket and notify interested parties about the change. It also delivers a notification in case it's removed from another basket in the process of moving.

So that's the new behavior.

There's still the same architectural smell: no method overrides in the subclass.

But we _could_ add methods related to moving eggs from one basket to another. And as a bonus it would make sense, too:

    #!swift
    class Egg: CoreDataEgg {
        
        override func moveToBasket(basket: Basket) {
            
            removeFromCurrentBasket()
            super.moveToBasket(basket)
            
            DomainEventPublisher.sharedInstance
                .publish(EggMovedToBasket(eggID: eggID, basket: basket.basketID))
        }
        
        private override func removeFromCurrentBasket() {
            
            guard let oldBasketID = basketID else {
                return
            }
            
            super.removeFromCurrentBasket()
            
            DomainEventPublisher.sharedInstance
                .publish(EggMovedToBasket(eggID: eggID, basket: basketID))
        }
        
        // ...
    }
    
    class CoreDataEgg: NSManagedObject {
        
        // Keep this one private at the Core Data level
        private func moveToBasket(basket: Basket) {
            self.basket = basket
        }
        
        private func removeFromCurrentBasket() {
            self.basket = .None
        }
        
        // ...
    }

This makes sense because `Egg` refers to `Basket`s via its `basketID` property. It doesn't (and for the sake of [encapsulating consistency boundaries][dddagg] shouldn't) have access to the other Domain Model object itself. The underlying association is provided by Core Data and merely an implementation detail.

Now the subclass delegates changing associations to its superclass. The smell uncovered a mix of responsibilities. Changing that was a smart move.

The other problems still apply: the public interface of `Egg` is cluttered with `NSManagedObject` stuff. But the situation is improving.

[doms]: /posts/2015/10/domain-model-core-data/
[crud]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[dddagg]: http://www.informit.com/articles/article.aspx?p=2020371&seqNum=4
