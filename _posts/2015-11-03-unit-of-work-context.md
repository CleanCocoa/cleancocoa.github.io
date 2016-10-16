---
title: Core Data Calls Them Contexts for a Reason
created_at: 2015-11-03 22:31:14 +0100
kind: worklog
tags: [ core-data ]
comments: on
---

It seems the [Unit of Work][uow] I envisioned is not the one I need. It seems Core Data calls `NSManagedObjectContext` a "context" for a reason. It doesn't suffice to set one up and pass it to every data store. At least not if you perform changes on background queues and with nested contexts.

So the best [I could come up with][last] was to expose the transactional context and add a completion block to schedule follow-up tasks until the end:

    #!swift
    func feedBiggestBanana() {
        let eventQueue = EventQueue()
    
        unitOfWork().execute({ context in
            let bananaRepo = CoreDataBananaRepository(context: context)
            let childRepo = CoreDataChildRepository(context: context)

            let child = childRepo.hungriestChild()
            let banana = bananaRepo.biggestBanana()
    
            child.eat(banana, eventQueue) // enqueue AteFood event
        }, completion: {
            DomainEventPublisher.publishFromQueue(eventQueue)
        })
    }

    clann Child {
        func eat(food: Food) { ... }
    }

It's not too bad to have an `NSManagedObjectContext` exposed to an Application Service. That's what they are made for.

But still ...

Instead, I could pass in something like a `UnitOfWorkConfiguration` to the block. It's a facade to the real repositories.

    #!swift
    protocol BananaRepository {
        func biggestBanana() -> Banana
    }
    
    protocol ChildRepository {
        func hungriestChild() -> Child
    }
    
    class CoreDataChildRepository: ChildRepository { ... }
    class CoreDataBananaRepository: BananaRepository { ... }
    
    class UnitOfWorkConfiguration {
        let transactionContext: NSManagedObjectContext
        
        init(transactionContext: NSManagedObjectContext) {
            self.transactionContext = transactionContext
        }
        
        lazy var bananaRepository = CoreDataBananaRepository(context: self. transactionContext)
        lazy var childRepository = CoreDataChildRepository(context: self. transactionContext)
    }
    
    extension UnitOfWorkConfiguration: ChildRepository {
        func hungriestChild() -> Child {
            return childRepository.hungriestChild()
        }
    }
    
    extension UnitOfWorkConfiguration: BananaRepository {
        func biggestBanana() -> Banana {
            return bananaRepository.biggestBanana()
        }
    }

This is pretty boring implementation-wise. It reminds me of what we did in Java. 

Even worse, though, it's focusing on an implementation problem. The resulting types are not proper objects. The facade has only fake responsibilities. It acts as something different while its real responsibility is to make client code easier: only a single `UnitOfWorkConfiguration` instance needs to be passed around instead of the real repositories. That's not the same quality of delegation as in a Book object having many Pages objects, delegating print-outs to the pages. I'd rather have a factory object to instantiate repositories with know the transactional context and create a new factory with each unit of work than a facade to deal with the code I myself created.

What's an object supposed to do? Currently, a unit of work can be executed. It reports errors. It collaborates with the managed object context. That's about it.

Can a unit of work not also prepare repositories? "Here, use that tool when you do your job!" I think that works out okay:

    #!swift
    let uow = unitOfWork()
    
    let bananaRepository = CoreDataBananaRepository()
    uow.prepare(bananaRepository)

    let childRepository = CoreDataChildRepository()
    uow.prepare(childRepository)
    
    uow.execute({ context in
        let child = childRepository.hungriestChild()
        let banana = bananaRepository.biggestBanana()

        child.eat(banana, eventQueue) // enqueue AteFood event
    }, completion: {
        DomainEventPublisher.publishFromQueue(eventQueue)
    })
    
Repositories (and other persistence mechanisms using `NSManagedObjectContext`s) can share a common protocol which the unit of work uses to set the context.

The other way around could work as well:

    #!swift
    let uow = unitOfWork()
    
    let bananaRepository = CoreDataBananaRepository(operateIn: uow)
    let childRepository = CoreDataChildRepository(operateIn: uow)
    
Here, the repositories would then obtain the current transactional context from the unit of work. I favor _Tell Don't Ask_ wherever I can to keep the flow of information easily comprehensible, so I don't feel too happy about this.

All this boils down to is that I'd like to open up a block and create repository instances in it and have them know about the ... _context._ That's when it hit me why the API designers may have called it `NSManagedObjectContext`. It's just everywhere. You can't easily box it away.

This is what I now have:

* Unit of work as sole source of truth about the current context.
* Repositories (and similar) cannot be set up during bootstrapping but have to be created with every new transaction.
* Consequently, Domain Services cannot hold on to repository instances but need to have them injected with every call. (This tradeoff is okay. Don't fight the framework too much.)
* Also, interactors (as data sources for view presenters) cannot operate in isolation but need guidance from, say, use case Application Services which take care of the unit of work setup.

In the end, there's only one source of the current context, and only one layer where these come from. From that follows that repository implementations are not to be referenced strongly in attributes but should be considered volatile.

Still not sure about how to get the context from the unit of work into related objects. The facade is out, although it'd be convenient.

I'll have to see where this is leading me. Drawing some diagrams now.

[uow]: /posts/2015/10/unit-of-work-core-data-transaction/
[last]: /posts/2015/11/core-data/
