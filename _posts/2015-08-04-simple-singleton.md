---
title: How to Write Pragmatic, Testable Singletons in Swift
created_at: 2015-08-04 08:15:06 +0200
kind: worklog
tags: [ singleton, testing, design-pattern ]
vgwort: http://vg08.met.vgwort.de/na/4768b56224a646719b439dc3fb1b7ce3
comments: on
---

Singletons have their use. I use two Singletons regularly in my projects: a [`DomainPublisher`][event] and a `ServiceLocator`. The latter is a registry of service objects which is global so the service objects don't have to be.

For practical use, most Singletons are overcomplicated or overly restrictive. Here's how I suggest you implement Singletons in your apps:

    #!swift
    class ServiceLocator {
        class var sharedInstance: ServiceLocator = ServiceLocator()
        
        func obtainSomeService() -> TheService {
            // ...
        }
    }

That's very simplistic and lacking a lot of the features Singletons usually have. Let me show you why those things don't matter.

The "correct" Singleton usually has these properties:

* it exposes a static shared instance, 
* which is preferably lazy-loaded,
* and usually constant once initialized.

In Swift, that has become quite easy to do thanks to the `let` keyword:

    #!swift
    class ServiceLocator {
        class let sharedInstance: ServiceLocator = ServiceLocator()
        
        // ...
    }

But now you can't provide replacements in your tests. That's pretty bad, since Singletons serve their shared instance or otherwise associated objects like object factories: the client code can simply ask for an instance.

If your Singleton cannot be replaced with a test double, obtaining objects through the Singleton has no advantage to creating objects directly.

Object initialization is impossible to test. That's why [Dependency Injection][di] is so popular a pattern. If your Singletons accessors are just as hard to test, you have the same problem.

Static constant shared instances are true to the pattern but don't help you code.

So you may use flexible Singletons: initialize once in production, but expose mutating methods for flexibility. This is what Objective-C enabled us to do. I wrote about that [before](/posts/2014/12/refactoring-singletons-in-ios/).

The result in Swift:

    #!swift
    private struct ServiceLocatorStatic {
        static var singleton: ServiceLocator? = nil
        static var onceToken: dispatch_once_t = 0
    }

    public class ServiceLocator {
    
        public init() { }
    
        public class var sharedInstance: ServiceLocator {
        
            if ServiceLocatorStatic.singleton == nil {
                dispatch_once(&ServiceLocatorStatic.onceToken) {
                    self.setSharedInstance(ServiceLocator())
                }
            }
        
            return ServiceLocatorStatic.singleton!
        }
    
        /// Reset the static `sharedInstance`, for example for testing
        public class func resetSharedInstance() {
        
            ServiceLocatorStatic.singleton = nil
            ServiceLocatorStatic.onceToken = 0
        }
    
        public class func setSharedInstance(instance: ServiceLocator) {
        
            ServiceLocatorStatic.singleton = instance
        }
        
        // ...
    }

Using `dispatch_once`, the shared instance is only created once, here even lazily. This implementation offers `resetSharedInstance()` for use during `tearDown()` to give all test cases a clean state.

[Uncle Bob made priorities clear][singleton]: tests trump encapsulation. Exposing `resetSharedInstance()` and `setSharedInstance(_:)` makes the Singleton testable. That's more important than having ideal encapsulation, because without tests, how do you know it does the right thing? Untestable code hurts your code quality.

But if that's our guiding principle, why insist on the `dispatch_once` dance? _Can't we trust our team not to be idiotic?_, Uncle Bob asks. If you're a single developer, it's even more silly to over-complicate your code in favor of "true" pattern adherence. 

Just make the shared instance  a static variable and get rid of all the other safeguarding mechanisms around Singletons.

    #!swift
    class ServiceLocator {
        class var sharedInstance: ServiceLocator = ServiceLocator()

        // ...
    }

This can be replaced during tests just as well (although now you need to keep track of the initial value to reset it later).

Keep in mind that once your Singletons become part of the public API, you cannot trust client code not to mess around with your objects. Then you'll need safeguarding mechanisms. But not any sooner.



[event]: /posts/2015/03/
[singleton]: http://blog.8thlight.com/uncle-bob/2015/06/30/the-little-singleton.html
[di]: https://en.wikipedia.org/wiki/Dependency_injection
