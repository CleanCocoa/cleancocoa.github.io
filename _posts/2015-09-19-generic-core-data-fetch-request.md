---
title: Save Yourself Casting Headaches with a Generic Fetch Request Object
created_at: 2015-09-19 15:02:12 +0200
kind: worklog
tags: [ core-data, swift, generics ]
comments: on
---

Swift 2 brings a lot of Cocoa API changes. For one, Core Data's `NSManagedObject`'s `executeFetchRequest(_:)` can throw, returns a non-optional on success, but still returns `[AnyObject]`. While refactoring existing code to use do-try-catch, I found the optional casting to `[MyEntity]` cumbersome.

So here's `CoreDataFetchRequest<T>`:

    #!swift
    import Foundation
    import CoreData

    enum CoreDataError: ErrorType {
        case InconsistentCoreDataFetchRequestResults
    }

    class CoreDataFetchRequest<T: NSManagedObject> {
    
        let fetchRequest: NSFetchRequest
    
        var predicate: NSPredicate? {
            get { return fetchRequest.predicate }
            set { fetchRequest.predicate = newValue }
        }
    
        convenience init(entityName: String) {
        
            self.init(fetchRequest: NSFetchRequest(entityName: entityName))
        }
    
        init(fetchRequest: NSFetchRequest) {
        
            self.fetchRequest = fetchRequest
        }
    
        func executeInContext(context: NSManagedObjectContext) throws -> [T] {
        
            let results = try context.executeFetchRequest(fetchRequest)
        
            guard let properResults = results as? [T] else {
                throw CoreDataError.InconsistentCoreDataFetchRequestResults
            }
        
            return properResults
        }
    }

I turned its interface the other way around: instead of extending `NSManagedObjectContext` to accept these, I call `executeInContext(_:)`. I like that better because most actions revolve around the fetch request more than the context. So the fetch request should be the "subject" here anyway.

You can use it like regular fetch requests, only without the if-let or forced casting to the expected entity:

    #!swift
    func bananaSizes() -> [BananaSize]? {
        
        let request = Banana.fetchRequest()
        let results: [Banana]
        
        do {
            
            results = try request.executeInContext(self.context)
        } catch let error as CoreDataError {
            
            switch error {
            case .InconsistentCoreDataFetchRequestResults: programmerError("Expected [Banana] but got mixed results.")
            }
            
            return .None
        } catch let error as NSError {
            
            self.coreDataErrorHandler.handle(error)
            return .None
        }
        
        return results.map() { $0.bananaSize }
    }

Where the entity looks like this:

    #!swift
    @objc(Banana)
    class Banana: NSManagedObject {
        
        static let entityName = "Banana"
        
        static func fetchRequest() -> CoreDataFetchRequest<Banana> {
            
            return CoreDataFetchRequest<Banana>(entityName: entityName)
        }
        
        static func fetchRequestForCountry(country: Country) -> CoreDataFetchRequest<Banana> {

            let request = fetchRequest()
            request.predicate = NSPredicate(format: "countryCode == %@", country.countryCode)
            
            return request
        }
        
        // ...
        
        @NSManaged var centimeterSize: Double
        
        var bananaSize: BananaSize {
            return BananaSize(cm: centimeterSize)
        }
    }

Honestly, I think in most cases you can safely ditch the `.InconsistentCoreDataFetchRequestResults` error and force the casting. Why should the results ever be mixed if you control the fetch requests? Better have a solid test harness to avoid weird mistakes in cases like this.

I am still finding my way around `throw` and friends. The readability of my code has worsened a bit after the Swift 2 conversion. It's time to find new patterns to handle that stuff.
