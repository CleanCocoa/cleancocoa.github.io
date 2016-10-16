---
title: How to Create Flexible Components with Two Levels of Service Objects
created_at: 2015-09-07 15:58:16 +0200
kind: worklog
tags: [ ddd, service, domain, use-case, core-data, transaction ]
vgwort: http://vg08.met.vgwort.de/na/94b88f45c55a4ba3805f3069217b2da4
comments: on
---

Again I was reminded about the value of service objects to encapsulate sequences in two places: the Domain, where business rules are kept, and in client code, where database management and transactions play a role. 

This distinction helps create clean code and deliver flexible components.

I am building a feature for the Word Counter in a separate "library" project until everything runs smoothly. Then I'll integrate the code into the main application. Since I consider this to be some kind of library, I thought I wouldn't need many service objects, but I was wrong.

The feature revolves around monitoring files. The thing with files is that their location may change or they can be deleted. Location-independent file URLs already work out of the box. And thanks to so-called bookmarks, you can persist self-updating file pointers in a database. Still, deletion will ruin everything. So there's a lot of edge cases to be considered.

When I tackled the sequence of updating word counts of all tracked files, I envisioned a type called `UpdateWordCounts`. Its `updateAll()` method would fetch all bookmarks from the database, turn these into URLs, and iterate over that collection: for each valid URL fetch the word count and store it.

That's a lot of stuff to do, so I ended up breaking this into two classes:

* `UpdateWordCount` (note the singular form!) stores the word count of a file at a given URL in the database.
* `UpdateWordCounts` (plural!) fetches all bookmarks and attempts the URL conversion. It delegates to `UpdateWordCount` to do something with valid URLs.

Note that its `UpdateWordCount`, not `WordCountUpdater`. By convention I name most service objects in a way which conveys the action involved. That's especially true for those service objects that encapsulate single use cases.

I thought these would be service objects outside the library and that the entry point at `UpdateWordCounts` (plural) is the use case. You know, stuff the client code will implement. 

In the end, the service objects worked with interfaces and didn't have to know anything about Core Data or the client's needs:

    #!swift
    public class UpdateWordCounts {

        let fileStore: FileStore
    
        public init(fileStore: FileStore) {
        
            self.fileStore = fileStore
        }
    
        public lazy var urlResolver: URLResolver = URLResolver()
        public lazy var missingURLHandler: HandlesMissingURL = 
            ServiceLocator.missingURLHandler
        public lazy var wordCountUpdater: UpdateWordCount = 
            UpdateWordCount(bookmarkRefresher: BookmarkRefresher(fileStore: self.fileStore))
    
        public func updateAll() {
        
            let bookmarks = fileStore.allBookmarks()
            let resolvedURLs = urlResolver.resolve(bookmarks)
        
            for missingURL in resolvedURLs.missingURLs {
                reportMissing(missingURL)
            }
        
            for validURL in resolvedURLs.validURLs {
                update(validURL)
            }
        }
    
        func reportMissing(missingURL: MissingURLInfo) {
        
            missingURLHandler.handleMissingURL(missingURL)
        }
    
        func update(urlInfo: URLInfo) {
        
            wordCountUpdater.update(urlInfo)
        }
    }

    public class UpdateWordCount {
    
        let bookmarkRefresher: BookmarkRefresher
    
        public init(bookmarkRefresher: BookmarkRefresher) {
    
            self.bookmarkRefresher = bookmarkRefresher
        }
    
        public lazy var monitorVendor: MonitorVendor = 
            ServiceLocator.monitorVendor
        public lazy var unmonitorableURLHandler: HandlesUnmonitorableURL = 
            ServiceLocator.unmonitorableURLHandler
    
        public func update(urlInfo: URLInfo) {
        
            let localURL = urlInfo.localURL
        
            localURL.scopedDo { URL in
            
                if urlInfo.isStale {
                    updateStaleBookmark(urlInfo)
                }
            
                if let monitor = monitorVendor.monitorFileAtURL(URL) {
                    monitor.fileChanged()
                } else {
                    unmonitorableURLHandler.handleUnmonitorableURLInfo(urlInfo)
                }
            }
        }
    
        func updateStaleBookmark(urlInfo: URLInfo) {
        
            bookmarkRefresher.refreshStaleBookmarkOfURLInfo(urlInfo)
        }
    }

Since the Core Data-backed concrete `FileStore` implementation requires that I save it periodically, I pondered if the `FileStore` protocol should have a `save()` method on it.

That'd satisfy the requirement short-term. But the code is so clean already that I wouldn't have to work any harder if I tried to avoid making the `FileStore` protocol dirty with implementation details.

That made me wonder: could `UpdateWordCounts` really be something like a Domain Service instead which would make me _enforce_ keeping Core Data-related code out of it?

I'm not trying to adhere to Domain-Driven Design for this small component. But the conventions and patterns are second nature to me already, so thinking in terms of DDD design patterns still works.

DDD taught me that there may be two layers of service objects in my code base, and that's just fine:

1. **Domain Services** which encapsulate sequences with Domain-specific rules.
2. **Application Services** which are direct clients of the domain and often represent use cases.

`UpdateWordCounts` doesn't encapsulate any particularly interesting sequence. It's made up of a two-step data retrieval and data conversion part and then delegates the results already. That's why I initially supposed this should be part of the client, not of the library.

I moved `UpdateWordCounts` and `UpdateWordCount` into the library group and ended up having nothing left in the sample app group to initiate updating all file records.

An Application Service object may have knowledge about the concrete infrastructure components involved -- Core Data, that is. An Application Service is the place where transactions are handled. And where `NSManagedObjectContext`s should be saved.

A proper translation of "transaction" to Core Data terms may be [Unit of Work][uow] pattern. A Unit of Work can use `performBlock(_:)` to bundle changes to the context and save the context afterwards:

    #!swift
    public class UnitOfWork {
    
        let managedObjectContext: NSManagedObjectContext
        let errorHandler: HandlesError
    
        public init(managedObjectContext: NSManagedObjectContext, 
            errorHandler: HandlesError) {
            
            self.managedObjectContext = managedObjectContext
            self.errorHandler = errorHandler
        }

        /// Asynchronously execute `closure` to sequentially perform transactions.
        public func execute(closure: () -> ()) {
            
            var error: NSError? = nil
        
            managedObjectContext.performBlock {
                closure()
            
                let success = self.managedObjectContext.save(&error)
            
                if !success {
                    self.errorHandler.handle(error)
                }
            }
        }
    }

Equipped with this little helper, the new Application Service becomes really simple:

    #!swift
    public class UpdateSchedule {
    
        let fileStore: FileStore
    
        public init(fileStore: FileStore) {
        
            self.fileStore = fileStore
        }
    
        public lazy var bookmarkRefresher: BookmarkRefresher = BookmarkRefresher(fileStore: self.fileStore)
        public lazy var updateWordCounts: UpdateWordCounts = UpdateWordCounts(bookmarkRefresher: self.bookmarkRefresher)
    
        public func execute() {
        
            unitOfWork().execute() {
            
                self.updateAllWordCounts()
            }
        }
    
        private func updateAllWordCounts() {
    
            let bookmarks = fileStore.allBookmarks()
        
            updateWordCounts.updateBookmarks(bookmarks)
        }
    }

So what's the benefit? What did I achieve?

* `UpdateSchedule` fetches data and performs a Unit of Work, including saving the Core Data context.
* `UpdateWordCounts` does the actual work. It doesn't need to obtain data anymore, so the public interface has changed a bit. The Application Service takes care of that now.
* `BookmarkRefresher` is tied to Core Data, too, and is now injected from an Application Service into the Domain Service. The Domain defined its, though, so it knows what's to be expected.

Not cramming everything into one service object was great. The tests are focused: I don't have to worry about the actual URL retrieval from bookmarks, asynchronous Unit of Work processing, and obtaining data from the store in one place and one test suite.

The tests are highly focused. That's nice. I have to do a bit more set up in tests of the Application Service because it accesses infrastructure components. But the Domain Service isn't cluttered by these requirements.

[uow]: http://martinfowler.com/eaaCatalog/unitOfWork.html
