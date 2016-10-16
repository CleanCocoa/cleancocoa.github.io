---
title: This is probably the nicest Swift Result enum ever
created_at: 2015-05-28 10:57:55 +0200
kind: worklog
tags: [ swift, control-flow, exception ]
comments: on
---

Until today, I had a regular-looking "fetch all" method on a Repository I am working with. It obtains all records from Core Data.

    #!swift
    public func records() -> [Record] {
        let fetchRequest = Record.fetchRequest()
        
        var error: NSError? = nil
        let results = managedObjectContext.executeFetchRequest(fetchRequest, error: &error)
        
        if results == nil {
            logError(error!, operation: "find existing records")
            postReadErrorNotification()
            assert(false, "error fetching records")
            return []
        }
        
        return results as! [Record]
    }

When I discovered the nice [Result enum by Rob Rix](https://github.com/antitypical/Result), things changed. I talked about using a Result enum [before](/posts/2015/02/functional-swift-exceptions/), but haven't used an implementation with so many cool additions.

Among my favorites is the `try` function. You pass it a block with a call to a Cocoa method which populates an `NSErrorPointer` on failure. `try` creates a `Result.Failure(NSError)` for you in return.

With that in place, the simple fetch method became even nicer:
    
    #!swift
    public func records() -> [Record] {
        let fetchRequest = Record.fetchRequest()
        
        let results = try { self.managedObjectContext.executeFetchRequest(fetchRequest, error: $0) }
        
        switch results {
        case let .Success(boxedValue): return boxedValue.value as! [Record]
        case let .Failure(boxedError):
            logError(boxedError.value, operation: "find existing records")
            postReadErrorNotification()
            assert(false, "error fetching records")
            return []
        }
    }

Boxing is annoying, and maybe a future Swift will make this obsolete. Until then, this works nicely and, to me, reads way better. I like that I don't need to handle an `error` variable myself. And I like that the words "Success" and "Failure" are included, so no more finding out which is the happy path.
