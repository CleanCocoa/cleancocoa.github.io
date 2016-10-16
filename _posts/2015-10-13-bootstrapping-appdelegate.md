---
title: How to Move Bootstrapping Code Out of AppDelegate
created_at: 2015-10-13 12:31:42 +0200
kind: worklog
tags: [ clean, viper, bootstrapping ]
shared_image: "blog/201509021531_clean-up-t.jpg"
comments: on
---

The test project I work on to develop a component of the [Word Counter][wcapp] faster sets up a lot of service objects during launch. I shaved off 60 lines of `AppDelegate` by introducing bootstrapping as a _thing_ to my code.

* A `Bootstrapping` component sets up itself as it pleases.
* A `Bootstrapped` collection tries to bootstrap new components. If the set-up succeeds, the collection grows.

This way you can prepare a list of `Bootstrapping` components and iterate over them to set up one after another.

The framework is very simple:

    #!swift
    enum BootstrappingError: ErrorType {
        case ExpectedComponentNotFound(String)
    }

    protocol Bootstrapping {
    
        func bootstrap(bootstrapped: Bootstrapped) throws
    }

    struct Bootstrapped {
    
        private let bootstrappedComponents: [Bootstrapping]
    
        init() {
        
            self.bootstrappedComponents = []
        }
    
        init(components: [Bootstrapping]) {
        
            self.bootstrappedComponents = components
        }
    
        func bootstrap(component: Bootstrapping) throws -> Bootstrapped{
    
            try component.bootstrap(self)
        
            return Bootstrapped(components: bootstrappedComponents.add(component))
        }
    
        func component<T: Bootstrapping>(componentType: T.Type) throws -> T {
        
            guard let found = bootstrappedComponents.findFirst({ $0 is T }) as? T else {
                throw BootstrappingError.ExpectedComponentNotFound("\(T.self)")
            }
        
            return found
        }
    }

`bootstrap(_:)` in a command which potentially grows the collection -- as long as no error is thrown.

`component(_:)` is a query to obtain an already bootstrapped component, indicating a dependency. If the sequence is out of order, this query method will throw an `.ExpectedComponentNotFound` error and the bootstrapping part of the app's launch sequence will not succeed. 

Note I haven't declared `bootstrappedComponents` as `var`. Neither have I made `bootstrap(:_)` mutating. That caused trouble when piping the bootstrapping with `reduce` like this:

    #!swift
    // Bootstrap the sequence of components
    func bootstrapped(components: [Bootstrapping]) throws -> Bootstrapped {

        return try components.reduce(Bootstrapped()) { bootstrapped, next in
    
            return try bootstrapped.bootstrap(next)
        }
    }

`bootstrapped(_:)` starts with an empty collection and sequentially tries to bootstrap every component, returning the complete collection in the end. This is like a for-each loop to add up numbers, only more terse:

    var result = Bootstrapped()
    for nextComponent in components {
        result = result.bootstrap(nextComponent)
    }
    return result

To keep the following example simple, none of the components take any arguments during initialization, though in production I pass a few preset values in.
    
    #!swift
    @NSApplicationMain
    class AppDelegate: NSObject, NSApplicationDelegate {
    
        // ...
        
        let components: [Bootstrapping] = [
            ErrorReportBootstrapping(),
            PersistenceBootstrapping(),
            DomainBootstrapping(),
            ApplicationBootstrapping()
        ]

        // Will be set during bootstrapping
        var showWindowService: ShowMainWindow!

        func applicationDidFinishLaunching(aNotification: NSNotification) {
    
            // Prepare everything
            bootstrap()
    
            // Then do something, like displaying the main window
            showWindow()
        }
        
        func bootstrap() {
            
            do {

                let bootstrapped = try bootstrapped(components)
                let application = try bootstrapped.component(ApplicationBootstrapping.self)
        
                showWindowService = application.showWindowService
            } catch let error as NSError {
                showLaunchError(error)
                fatalError("Application launch failed: \(error)")
            }
        }

        func bootstrapped(components: [Bootstrapping]) throws -> Bootstrapped {
    
            return try components.reduce(Bootstrapped()) { bootstrapped, next in
        
                return try bootstrapped.bootstrap(next)
            }
        }
        
        func showLaunchError(error: NSError) {
    
            let alert = NSAlert()
            alert.messageText = "Application launch failed. Please report to support@christiantietze.de"
            alert.informativeText = "\(error)"
    
            alert.runModal()
        }
        
        func showWindow() {
    
            showWindowService.showWindow()
        }
    }


Error reporting, for example, is very simple. That's by design since it's so fundamental.

    #!swift

    class ErrorReportBootstrapping: Bootstrapping {
    
        let errorHandler = ErrorHandler()
        let invalidDataHandler = InvalidDataHandler()
    
        func bootstrap(bootstrapped: Bootstrapped) throws {
        
            ErrorAlert.emailer = TextEmailer()
        
            // Add objects to a global registry of services used application-wide.
            ServiceLocator.sharedInstance.errorHandler = errorHandler
            ServiceLocator.sharedInstance.missingURLHandler = invalidDataHandler
            ServiceLocator.sharedInstance.fileNotFoundHandler = invalidDataHandler
        
        }
    }

Objects can declare dependencies by querying the `Bootstrapped` collection:

    #!swift
    import Foundation

    class DomainBootstrapping: Bootstrapping {
    
        var bananaPeeler: BananaPeeler!
    
        func bootstrap(bootstrapped: Bootstrapped) throws {
          
            // may throw .ExpectedComponentNotFound("PersistenceBootstrapping")
            let persistence = try bootstrapped.component(PersistenceBootstrapping.self)
        
            bananaPeeler = BananaPeeler(bananaStore: persistence.bananaStore)
        }
    }

And then `ApplicationBootstrapping` will be able to fetch `bananaPeeler` from the previously set-up `DomainBootstrapping` and use it to delegate from the event handler for a window controller it sets up, say.

Some of the `Bootstrapping` components do a similar job to VIPER's wireframes which prepare interactor, presenter, and view. The whole `Bootstrapped` collection wraps the set-up of wireframes, building what some (including me, usually) call an `AppDependencies` object, holding the Core Data stack and other things at once, sequentially.

I like that the `Bootstrapping` components work in isolation and group similar concepts. That works better than `// MARK`'ing groups of concepts in a single `AppDependencies` container.

[wcapp]: http://wordcounterapp.com

---

Photo Credit: [Please Clean Up Your Mess](https://www.flickr.com/photos/allen_goldblatt/194069411) by [Allen Goldblatt](https://www.flickr.com/photos/allen_goldblatt/). License: [CC-BY-2.0](https://creativecommons.org/licenses/by/2.0/).
