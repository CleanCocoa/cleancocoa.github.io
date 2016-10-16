---
title: "Put Usage of a CoreDataFetchRequest out of Your Code and Into ... Where Exactly?"
created_at: 2015-10-09 11:12:51 +0200
kind: worklog
tags: [ core-data, swift, generics, repository, protocol ]
comments: on
---

I created a [generic CoreDataFetchRequest][fr] a while ago and love it.

It casts the results to the expected values or throws an error if something went wrong, which it shouldn't ever, logically -- but the Core Data API currently returns `AnyObject` everywhere, so I have to deal with that.

I have found three ways to use it. I'd like to know which one you prefer.

1. Using the `CoreDataFetchRequest` directly
2. Using a `CoreDataStore` to encapsulate using the requests
3. Extending the `CoreDataFetchRequest` with behavior

This is the managed object I'm going to query:

    #!swift
    @objc(Banana)
    class Banana: NSManagedObject {
    
        static let entityName = "Banana"
    
        static func fetchRequest() -> CoreDataFetchRequest<Banana> {
        
            return CoreDataFetchRequest<Banana>(entityName: entityName)
        }
    }


## 1: Using CoreDataFetchRequest as a wrapper

Here's how it's used directly:
    
    #!swift
    func getAllBananas(context: NSManagedObjectContext) -> [Banana] {
        
        let request = Banana.fetchRequest()
        let results: [Banana]
        
        do {
            results = try request.executeInContext(self.context)
        } catch let error as CoreDataError {
    
            switch error {
            case .InconsistentCoreDataFetchRequestResults: 
                fatalError("Expected [Banana] but got mixed results.")
            }
    
            return []
        } catch let error as NSError {
    
            fatalError("Core Data error: \(error)")
            return []
        }
        
        return results
    }

* It's long.
* It repeats every time I use a `CoreDataFetchRequest`.
* `do-try-catch` is ugly.

So I thought maybe I should wrap that up. `CoreDataFetchRequest` clients are "stores" or "repositories". So I created a protocol for them.

## 2: Hide CoreDataFetchRequest in a CoreDataStore

Using a `CoreDataStore`-compliant object works like that:

    #!swift
    protocol BananaReader {
        func allBananas() -> [Banana]
    }
    
    class CoreDataBananaReader {
        let context: NSManagedObjectContext

        init(context: NSManagedObjectContext) {
            self.context = context
        }
    }
    
    extension CoreDataBananaReader: CoreDataStore { }
    
    extension CoreDataBananaReader: BananaReader {
        
        func allBananas() -> [Banana] {
            
            let request = Banana.fetchRequest()
            return resultOfFetchRequest(request)
        }
    }

Simplified, this is what the `CoreDataStore` looks like:

    #!swift
    protocol CoreDataRepository {
    
        var context: NSManagedObjectContext { get }

        func resultOfFetchRequest<T>(request: CoreDataFetchRequest<T>) -> [T]
        func firstResultOfFetchRequest<T>(request: CoreDataFetchRequest<T>) -> T?
    }

    extension CoreDataRepository {
    
        func resultOfFetchRequest<T>(request: CoreDataFetchRequest<T>) -> [T] {
        
            let results: [T]
        
            do {
                results = try request.executeInContext(context)
            } catch let error as CoreDataError {
            
                switch error {
                case .InconsistentCoreDataFetchRequestResults: 
                    fatalError("Expected [\(T.self)] but got mixed results.")
                }
            
                return []
            } catch let error as NSError {
                fatalError("Core Data error: \(error)")
                return []
            }
        
            return results
        }
    
        func firstResultOfFetchRequest<T>(request: CoreDataFetchRequest<T>) -> T? {
        
            return resultOfFetchRequest(request).first
        }
    }

* The `allBananas()` implementation is down to 2 lines from 20.
* Protocol extensions feel weird because now the `CoreDataBananaReader` inherits behavior, which is a different kind of abstraction than subclasses. It's a new concept to me.
* Two different concrete stores share the `CoreDataStore` protocol. They look alike and "are" alike.
* The request doesn't have real behavior. The method asks for a request and uses it. There's the thing and its handler: request and store.

So maybe put that stuff into the `CoreDataFetchRequest` itself?

## 3: Extending CoreDataFetchRequest to do something on its own

Using an extension is short and simple. The basic reader protocol stays the same:

    #!swift
    protocol BananaReader {
        func allBananas() -> [Banana]
    }
    
    class CoreDataBananaReader {
        let context: NSManagedObjectContext

        init(context: NSManagedObjectContext) {
            self.context = context
        }
    }
    
    extension CoreDataBananaReader: BananaReader {
        
        func allBananas() -> [Banana] {
            
            return Banana.fetchRequest().allResultsInContext(context)
        }
    }

Scratch the `CoreDataStore` protocol and instead add this:

    #!swift
    extension CoreDataFetchRequest {
    
        func allResults(context context: NSManagedObjectContext) -> [T] {
        
            let results: [T]
        
            do {
                results = try executeInContext(context)
            } catch let error as CoreDataError {
            
                switch error {
                case .InconsistentCoreDataFetchRequestResults: 
                    fatalError("Expected [\(T.self)] but got mixed results.")
                }
            
                return []
            } catch let error as NSError {
                fatalError("Core Data error: \(error)")
                return []
            }
        
            return results
        }    
    }

* It becomes a one-liner.
* No passing around of values. The request itself has behavior. That's always a better encapsulation to me.
* A `CoreDataBananaReader` and, say, a `CoreDataAppleReader` don't have anything in common, since they don't implement a protocol like `CoreDataStore` anymore. They just look and work alike. (Maybe that's a real problem only for Java folks but not for us Cocoa devs. Smalltalk for the win!)

So, what do you prefer? Any arguments pro 2 and contra 3?

[fr]: /posts/2015/09/generic-core-data-fetch-request/
