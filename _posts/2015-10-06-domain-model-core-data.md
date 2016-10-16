---
title: Expressive Domain Model, Core Data, and You
created_at: 2015-10-06 09:39:23 +0200
kind: worklog
tags: [ core-data, ddd, model, domain ]
comments: on
---

I've written [a whole book](https://leanpub.com/develop-mac-apps-clean-architecture-swift) on the topic of architecting a Mac app using Core Data. Today I found an approach which doesn't add lots of overhead and which also doesn't clutter everything everywhere.

It's pretty simple and involves Swift.

Here's the steps you need to take:

1. Create a `NSManagedObject` subclass for your entity. I like to prefix mine with "CoreData". Make all `@NSManaged` properties _private_, at least the setter. That's important to avoid clients changing every possible attribute of your model.
2. If the stored property is a `String` or `NSData` representation of something else, for example, mark the whole property private and provide a public getter which wraps the representation in the real thing.
3. In the same file, create an internal or public domain object with the appropriate name as a subclass of your managed object.

Since I like to put some static convenience methods on my managed objects, I am currently considering to use double-dispatch 

Here's a very simple example.

First, the value objects which are part of the domain:

    #!swift
    // MARK: Value Objects - may live in a separate file
    
    enum EggSize: Int {
        case Small = 1
        case Medium
        case Large
        case XLarge
    }
    
    struct EggID {
        
        let identifier: String
        
        init() {
            self.identifier = NSUUID().UUIDString
        }

        init(identifier: String) {
    
            self.identifier = NSUUID(UUIDString: identifier)!.UUIDString
        }
    }
    
    /// Contains all data necessary to create a new Egg
    struct NewEgg {
        let eggID: EggID
        let size: EggSize
    }

You see where this is going, don't you?
    
    #!swift
    // MARK: Domain Object
    
    @objc(Egg)
    class Egg: CoreDataEgg {
        
        var eggID: EggID {
            return EggID(identifier: eggIdentifier)
        }
        
        var size: EggSize {
            return EggSize(rawValue: eggSize)
        }
        
        // Now this doesn't make sense. But I needed to prove a point quickly :)
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
    }

Now `Egg`, which is exposed to clients, has a proper public API. It still inherits a lot of the `NSManagedObject` stuff, of course, but when you look at its interface, you can see what you should be doing with it.

That's a lot more than you'd know if you used `CoreDataEgg` directly with public or internal setters for the properties.

An expressive domain starts where you don't have to guess. It sports meaningful methods which reveal your intent. In that sense, `changeSize(_:)` reveals that eggs are intentionally resizable. It's arguably a better place than overriding the setter to publish events, too. 

_Don't fight the framework_, they say. Core Data is awesome. But I still don't want to have its internals linger everywhere.

Caveats (will add to the list as I encounter any):

* Using Core Data managed objects as domain objects makes testing harder: you can't create instances easily, like `Egg()`, but have to use `NSEntityDescription` instead.
* Initializer inheritance. You can't get rid of `init(entity:,insertIntoManagedObjectContext:)`. (2015-10-08)

I'm trying this approach for now and keep you updated.

**Update 3 hours later:** The `Egg` domain object has to subclass `CoreDataEgg`, of course, which makes the public interface of `Egg` "dirty." so far, it's the best trade-off between functionality and conceptual purity I've encountered. Simple CRUD models without much behavior don't need all of this -- you can better decouple the domain objects from the Core Data managed objects.
