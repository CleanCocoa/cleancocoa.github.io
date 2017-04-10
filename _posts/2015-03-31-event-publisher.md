---
title: Painless Event Delivery With a Custom Publish-Subscribe Infrastructure
created_at: 2015-03-31 11:14:33 +0200
kind: worklog
tags: [ event-sourcing, domain-event ]
image: 201503311115_box-t.jpg
vgwort: http://vg08.met.vgwort.de/na/addba93e665b41b4abdaac6c6adbed4c
comments: on
code: swift
---

In my apps, I have used a custom event delivery mechanism on top of `NSNotificationCenter` with great success: I send a lot more events and consequently decouple parts of my code more easily. Putting your own objects around the Cocoa foundation can help use them more often. I want to show you how my `EventPublisher` works, and why custom event types make me send a lot more events than when I only had `NSNotification`s.

Creating your own `EventPublisher` is a good answer to the question on how to test `NSNotificationCenter` usage, too. You don't test classes you don't own -- instead, you wrap them in your own objects. An `EventPublisher` is a rather thin wrapper around `NSNotificationCenter`, but still it opens up the possibility to rethink event-driven parts of your system.

Among others, these are the benefits:

1. Custom event Value Objects **make sending events _safe_**. You can guard against invalid or incomplete data. Protect the invariants of events and let the compiler spot misuse after you changed the required data. This amounts to even less tests needed.
2. Custom event objects are created quickly and are **more readable** than hacking together sending an event with `userInfo` and the like.
3. Event publishers may use different `NSNotificationCenter` instances and communicate more clearly: **one queue per publisher.**
4. Event publishers can take care of **picking an appropriate dispatch queue** for your (sub)system. Have yourself a publish/subscribe-subsystem which updates the view and operates on the main queue exclusively.

We can decouple command execution from response handling through events. The benefit to directly sending messages as methods is that we don't have to know the interested parties. Conversely, we don't have to design command callees to know how their caller handles the result of the command.

Even if in fact only one subscriber exists per message, the benefit of hiding its existence from the sender can be worth the effort. We don't design the sender to accommodate the receiver and increase coupling implicitly. That's what _informational messages_ are good for.

In a previous post, I sketched [how to react to changes of command execution using Domain Events][split]. There, I've used an `EventPublisher` class in the sample code. The actual implementation of this class was left as an "exercise to the reader," so to say. 

For the impatient: **see [the full source on GitHub][publisher].**

It's part of the sample code of my book about writing software for the Mac with XPC Service backends, [_Creating Multi-Process Mac Applications_][book].

[book]: https://leanpub.com/create-multi-process-mac-apps
[split]: /posts/2015/03/split-command-return-value-events/

## Improving `NSNotificationCenter`'s Capabilities

{% include figure.md src="/assets/blog/2015/201503311115_box.jpg" alt="child in a box" caption="Photo Credit: <a href=\"https://www.flickr.com/photos/markjsebastian/4217877353/\">\"Special Delivery (#100888)\"</a> by <a href=\"https://www.flickr.com/photos/markjsebastian/\">Mark Sebastian</a> on flickr. License: <a href=\"https://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA 2.0</a>" %}

`NSNotificationCenter` spoils us because it just works. The only thing you have to take care of is sending a notification of the right `name` with sufficient `userInfo`. This is error-prone, because only in runtime will we find whether we forgot to add an expected key-value-pair to the `userInfo` dictionary.

Wrapping the checks for `name` and `userInfo` in custom classes will help fix that. This way, the compiler will help us create events and react to them. One reason less for your program to fail.

I'd go as far to say only custom [Value Objects](http://martinfowler.com/bliki/ValueObject.html) make `NSNotificationCenter` useful for use in the first place.

### Example 1: Sending Events

This is the example from the [last post][split] on the topic:

```swift
extension Canvas {
    func save(toFile file: File) {
       // perform file saving of the canvas' data
   
       if (didFail) {
           EventPublisher.sharedInstance.publish(
               SavingCanvasFailed(canvasId: self.canvasId))
           return
       }

       EventPublisher.sharedInstance.publish(
           CanvasSaved(canvasId: self.canvasId))
   }
}
```

There are two event types:

1. `CanvasSaved`
2. `SavingCanvasFailed`

They are phrased in past tense because informational messages are sent after the fact they inform about. `SavingCanvasFailed` could be called `CanvasSavingFaild`, too, if you like. I like to put the verb in the front.

Both event types take a single parameter in this case. `canvasId` is a read-only public attribute of the object and will be used by subscribers to react to the event accordingly. If your event types take more parameters to sufficiently spell out the change in the system, great! Then your events just got even more useful.

Talking about subscribers, let us look at subscription next before we look at `EventPublisher`.

### Example 2: Receiving Events

From the same post, here's the subscriber code:

```swift
extension CanvasController {
    let canvasCollection = CanvasCollection()
    
    init() {
        // ...
        subscribeToCanvasEvents()
    }

    var savingFailedSubscription: Subscription!
    var savingSucceededSubscription: Subscription!
    
    func subscribeToCanvasEvents() {
        let publisher = EventPublisher.sharedInstance
        savingFailedSubscription = publisher.subscribeTo(SavingCanvasFailed.self) {
            [unowned self] event in
        
            // notify user about failure and prompt 
            // for picking another file
            if shouldTryAgain {
                canvas = self.canvasCollection.canvasWithId(event.canvasId)
                self.saveCanvas(canvas)
            }
        }
    
        savingSucceededSubscription = publisher.subscribeTo(CanvasSaved.self) {
            [unowned self] event in
        
            // for example display subtle success message in the 
            // corner of the screen
        }
    }
}
```

`subscribeToCanvasEvents` initializes two attributes of the subscriber, called "subscriptions". In the attached blocks, the subscriber can react to the events.

A `Subscription` is a wrapped `NSObjectProtocol` instance. You obtain `NSObjectProtocol` through calling `addObserverForName` on a `NSNotificationCenter`. Instead of the subscriber adding itself to the notification center via `addObserver(_:, name:, object:)`, it obtains a dedicated subscriber object. Even this subscriber object has to be removed from the notification center using `removeObserver(_:)`. That's what `Subscription` does upon `deinit`:

```swift
class Subscription {

    let observer: NSObjectProtocol
    let eventPublisher: EventPublisher

    public init(observer: NSObjectProtocol, eventPublisher: EventPublisher) {
        self.observer = observer
        self.eventPublisher = eventPublisher
    }

    deinit {
        eventPublisher.unsubscribe(observer)
    }
}
```

This way, the "real" subscriber (`CanvasController` in this case) doesn't have to worry about managing subscriptions. When `CanvasController` is deallocated, its subscriptions will be, too, and thus they will remove themselves from the notification center through `EventPublisher.unsubscribe(_:)`.

Now we know how the straight-forward sending works, and we know how to create subscriptions.

## Creating `EventPublisher` to Enhance `NSNotificationCenter`

To create events, here's the `Event` protocol which specifies the shared bare minimum of every event:

```swift
typealias UserInfo = [NSObject : AnyObject]

protocol Event {
    /// The `EventType` to identify this kind of DomainEvent.
    class var eventType: EventType { get }

    init(userInfo: UserInfo)
    func userInfo() -> UserInfo
    func notification() -> NSNotification
}
```

### The `CanvasSaved` Event Implementation

```swift
enum EventType: String {
    case CanvasSaved = "Canvas Saved"
    case SavingCanvasFailed = "Saving Canvas Failed"
    
    var name: String {
        return self.rawValue
    }
}

struct CanvasSaved: Event {
    static var eventType: EventType {
        return EventType.CanvasSaved
    }

    let canvasId: CanvasId

    init(canvasId: CanvasId) {
        self.canvasId = canvasId
    }

     init(userInfo: UserInfo) {
        let canvas = userInfo["canvas"] as UserInfo
        let canvasId = canvas["id"] as NSNumber
    
        self.init(canvasId: canvasId)
    }

    func userInfo() -> UserInfo {
        return [
            "canvas": [
                "id": canvasId.number
            ]
        ]
    }

    func notification() -> NSNotification {
        return NSNotification(name: self.dynamicType.eventType.name, object: nil, userInfo: userInfo())
    }
}
```

The `EventType` enum is basically there just to make notification names unique.[^refl] I don't use an enum-to-class lookup during subscription, but pass in the type itself via `subscribe(CanvasSaved.self)`. You may want to change that, though. 

Note that I added another initializer. Instances can be created in two ways: 

1. `CanvasSaved(canvasId: /*...*/)`, which we'll use and type when we send an event, and
2. `CanvasSaved(userInfo: ["canvas" : ["id" : "..."]])`, which we'll never invoke manually but which is used to re-create an event object from notification.

Sadly, we have to duplicate the `notification` method in every event, because Swift doesn't (currently) allow `anEvent.dynamicType` if `anEvent` is of protocol type `Event`. `dynamicType` works on concrete class types only.

There are other ways, but I found this the least dislike-able.


[^refl]: I prefer the explicit `EventType` enum over the rudimentary reflection capabilities of Swift via `_stdlib_getTypeName`. The latter would return something like `_TtC13__lldb_expr_014CanvasSaved`.



### The Actual `EventPublisher` and its `EventSubscription`

`EventPublisher` is a wrapper around sending `NSNotification` objects through a common `NSNotificationCenter`. It doesn't have to be the `defaultCenter`, though, so you can run multiple publishers next to each other.

The nice thing about this is that you can replace either the `EventPublisher` with a mock or stub object during unit tests, or create per-test `NSNotificationCenter` objects to verify actual event delivery.

At its core, it looks like this:

```swift
class EventPublisher {
    let notificationCenter: NSNotificationCenter
    
    convenience init() {
        self.init(notificationCenter: NSNotificationCenter.defaultCenter())
    }

    init(notificationCenter: NSNotificationCenter) {
        self.notificationCenter = notificationCenter
    }
    
    func publish(event: Event) {
        notificationCenter.postNotification(event.notification())
    }
    
    func subscribe<T: DomainEvent>(eventKind: T.Type, 
        usingBlock block: (T!) -> Void) -> EventSubscription {
        
        // Pick a different default queue if you want async event processing
        let mainQueue = NSOperationQueue.mainQueue()

        return self.subscribe(eventKind, queue: mainQueue, usingBlock: block)
    }

    func subscribe<T: DomainEvent>(eventKind: T.Type, 
        queue: NSOperationQueue, 
        usingBlock block: (T!) -> Void) -> EventSubscription {
        
        let eventType: EventType = T.eventType
        let observer = notificationCenter.addObserverForName(eventType.name, object: nil, queue: queue) {
            notification in
    
            let userInfo = notification.userInfo!
            let event: T = T(userInfo: userInfo)
            block(event)
        }
        
        return EventSubscription(observer: observer, eventPublisher: self)
    }
    
    func unsubscribe(subscriber: AnyObject) {
        notificationCenter.removeObserver(subscriber)
    }
}
```

The full source is [available online][publisher], including using the Singleton pattern.

Subscribing is the most interesting part of this little publisher.

I never used `addObserverForName(_:,object:,queue:)` much in the past. In case you didn't either: it basically creates an observer object of type `NSObjectProtocol` which is tied to the notification parameters so you don't have to call the verbose `removeObserver(_:,name:,object:)` later on to get rid of it. It suffices to simply use `removeObserver(_:)`.

`NSObjectProtocol` takes a block, which is great to use. If the notification is triggered, the block is invoked and an `NSNotification` is passed in. From this, we can re-create an appropriate Domain Event object.

As I said above already, the resulting `EventSubscription` merely takes care of unsubscribing automatically when its reference is nilled out:

```swift
class EventSubscription {
    let observer: NSObjectProtocol
    let eventPublisher: EventPublisher

    init(observer: NSObjectProtocol, eventPublisher: EventPublisher) {
        self.observer = observer
        self.eventPublisher = eventPublisher
    }

    deinit {
        eventPublisher.unsubscribe(observer)
    }
}
```

## Conclusion

Small custom types make much of a difference.

* `Subscription` is just an auto-unsubscribing `NSObjectProtocol`, but this already takes much of the burden of notifications away.
* `EventPublisher` mostly wraps `NSNotificationCenter` and hides this Cocoa class from your code. Since it's ours, we can replace it during tests easily.

If you use events to drive user interface change, dispatch events to subscribers from the main queue within `EventPublisher`. If you don't, use another queue. The publisher can take care of this much of multi-threading.

To create new event types, we need to perform three steps:

1. Add a new `Event` protocol implementation;
2. add an `EventType` entry for the name and use it from within the new type, of course; and finally
3. subscribe to the event and keep the subscription around.

Implementing an actual `Event` requires some boilerplate code. I'd like to eliminate that, and I guess I will end up with a slightly different approach in the future. The `EventType` enum could be used for subscriptions, or I could get rid of it completely.

If you'd like to try [event sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) instead of persisting snapshots of data, this is a good place to get started.

* **Have you created your own `NSNotificationCenter` wrappers in the past?**
* What did they look like?

[publisher]: https://github.com/DivineDominion/mac-multiproc-code/blob/master/RelocationManagerServiceDomain/DomainEventPublisher.swift
