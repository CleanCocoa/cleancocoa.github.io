---
title: Observing Enums in Swift 1.2 with KVO
created_at: 2015-05-13 08:45:21 +0200
kind: worklog
tags: [ swift, kvo ]
comments: on
code: swift
---

Swift 1.2 brought very nice additions to the functionality of the language. Among one of my favorites is that we can now use [Key-Value Observing][kvo] on `enum` attributes.

Here's an example. Capacities of container are pre-set in our domain, so it makes sense to create an enum to represent the allowed values:
    
    #!swift
    public enum Capacity: Int {
        case Small = 5
        case Medium = 10
        case Large = 20
    }

Until Swift 1.2 added support to export enums to Objective-C's `NS_ENUM`, a container wouldn't be able to observe such a property using KVO because it could not have been represented in Objective-C.

To make this work, you had to for example add another property of type `NSNumber` and keep it in sync with the enum via Swift's native `didSet` property observer:

    #!swift
    public class Container: NSObject {
        // ...    
        public dynamic private(set) var capacityRaw : NSNumber! = 0
    
        public var capacity: Capacity {
            didSet {
                self.convertCapacity()
            }
        }
    
        func convertCapacity() {
            capacityRaw = NSNumber(integer: capacity.rawValue)
        }
    
        public init(capacity: Capacity) {
            self.capacity = capacity
            super.init()

            // Call this once manually because `didSet` won't be invoked
            // during initialization.
            convertCapacity()
        }
    }

With Swift 1.2, workarounds such as these became obsolete. KVO works with enums directly once you annotate the enum with `@objc`:

    #!swift
    @objc public enum Capacity: Int {
        case Small = 5
        case Medium = 10
        case Large = 20
    }
    
    public class Container: NSObject {        
        // ...
        public dynamic private(set) var capacity: Capacity
    
        public init(capacity: Capacity) {
            self.capacity = capacity
            // No explicit call to `super.init()` necessary anymore
        }
    }

This is much nicer. Neither do we have to ensure the "raw" `NSNumber` stays in sync (and thus complicate our test suite), nor do we clutter the public interface of the class with seemingly redundant information.

[kvo]: http://nshipster.com/key-value-observing/
