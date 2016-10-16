---
title: Transactions and Rolling Back Changes in Core Data with UnitOfWork
created_at: 2015-10-12 16:13:26 +0200
kind: worklog
tags: [ transaction, core-data ]
comments: on
---

The [Unit of Work pattern][uow] is a code equivalent to database transactions: when it completes, changes are persisted; when something fails, changes are rolled back.

That's handy to perform a set of changes and have them saved. Sooner or later during app development using Core Data, you may ask: when should I save? When is the best point in time?

The rough answer is: whenever the user completed a task.

That means you should not save intermediate changes in a modally presented view controller scene which can be cancelled. Cancelling equals discarding all changes made in the form.

In most of my own apps, the answer is more specific: during most use case objects. I architect most apps around use cases, represented as small service objects. (_Application Services_, to be exact.) They perform a user-initiated change and save the result. They're the sole users of Units of Work. View controllers merely delegate user interactions to event handlers; the action itself is contained in these services.

In Core Data, you can achieve discarding of changes to managed objects with [child managed object contexts][cmocs]. Just get rid of the context.

Handling this manually introduces repetition, though. I'm quite content with code like this:

    #!swift
    // Application Service
    class RemoveFile {
    
        let fileStore: FileStore
        
        init(fileStore: FileStore) {
            
            self.fileStore = fileStore
        }
    
        func removeFiles(fileIDs: [FileID]) {
        
            guard !fileIDs.isEmpty else {
                return
            }
        
            unitOfWork().onError({ fatalError($0) }) // Report failure to user
                .execute({ try fileIDs.forEach(self.removeFile) })
        }
    
        private func removeFile(fileID: FileID) throws {
    
            do {
                try fileRemovalService().removeFile(fileID)
            } catch let error as NSError {
                logError("Unexpected error XYZ")
                
                throw error
            }
        }
        
        // Override this factory method during tests to inject a mock
        func fileRemovalService() -> FileRemoval {
        
            // A Domain Service adhering to business rules to 
            return FileRemoval(fileStore: fileStore)
        }
    }
    
* The real transaction spans a set of entities, so failure in entity number `N` should roll back changes to entities `0...N-1`. That's why the Unit of Work wraps the loop.
* To roll-back changes, simply throw during `execute`'s block.
* Conversely, re-throw errors that pop up during execution.

For an Application Service, this is very straightforward. That's why I picked the example.

The actual _Unit of Work_ looks like this:

    #!swift
    let mainContext: NSManagedObjectContext = ...
    
    func unitOfWork() -> UnitOfWork {
        return UnitOfWork(parentManagedObjectContext: mainContext)
    }
    
    enum UnitOfWorkError: ErrorType {
        /// Error raised during `performBlock` of the execute closure.
        case ExecutionError(NSError)
    
        /// Error raised when the temporaty transactional context cannot be saved.
        case CoreDataTransactionError(NSError)
    
        /// Error raised when the main context cannot be saved after the transaction.
        case CoreDataError(NSError)
    }

    class UnitOfWork {
    
        let parentManagedObjectContext: NSManagedObjectContext
        let managedObjectContext: NSManagedObjectContext
    
        convenience init(parentManagedObjectContext: NSManagedObjectContext) {
        
            precondition(!IsRunningTests, "Use designated initializer during tests")
        
            let context = NSManagedObjectContext(concurrencyType: .PrivateQueueConcurrencyType)
            context.parentContext = parentManagedObjectContext
        
            self.init(parentManagedObjectContext: parentManagedObjectContext, managedObjectContext: context)
        }
        
        init(parentManagedObjectContext: NSManagedObjectContext, managedObjectContext: NSManagedObjectContext) {
        
            self.parentManagedObjectContext = parentManagedObjectContext
            self.managedObjectContext = managedObjectContext
        }
    
        var errorHandler: ((UnitOfWorkError) -> Void)?
    
        func onError(handleError: (UnitOfWorkError) -> Void) -> UnitOfWork {
        
            errorHandler = handleError
        
            return self
        }
        
        func execute(closure: () throws -> Void) {
        
            managedObjectContext.performBlock {
            
                do {
                    try closure()
                } catch let error as NSError {
                    self.errorHandler?(UnitOfWorkError.ExecutionError(error))
                    return
                }
        
                do {
                    try self.managedObjectContext.save()
                } catch let error as NSError {
                    self.errorHandler?(UnitOfWorkError.CoreDataTransactionError(error))
                    return
                }
            
                self.parentManagedObjectContext.performBlock() {
                
                    do {
                        try self.parentManagedObjectContext.save()
                    } catch let error as NSError {
                        self.errorHandler?(UnitOfWorkError.CoreDataError(error))
                    }
                }
            }
        }
    }

When saving to the transaction's `managedObjectContext` succeeds, the changes are effectively propagated to the parent store which has to be saved right afterwards, too.

I'd rather have the `UnitOfWork` throw errors itself instead of using the kind of weird `errorHandler`. But `performBlock` is executed asynchronously, which is useful, so there's no deal-breaking, non-blocking way to capture errors of the block and throw them from `execute` again. 

I have unit tests for this, too, in case you're interested to use it in your app.

[uow]: http://martinfowler.com/eaaCatalog/unitOfWork.html
[cmocs]: https://www.cocoanetics.com/2012/07/multi-context-coredata/
