---
title: Event Publisher Revision 1
created_at: 2015-04-10 10:42:00 +0200
kind: worklog
tags: [ domain-event ]
comments: on
preview: fulltext
code: swift
---

I have imported my latest [event publishing][event] code into the Word Counter. Since the `DomainEventType` enum didn't provide much value besides obtaining a notification name, I decided to get rid of it and obtain a name by another means.

In the current iteration (see [the Gist][gist]), the event protocol reveals a static `eventName`. I also removed the duplicated notification factory from concrete events:

    #!swift
    import Foundation
  
    public typealias UserInfo = [NSObject : AnyObject]
  
    public protocol DomainEvent {
        class var eventName: String { get }
  
        init(userInfo: UserInfo)
        func userInfo() -> UserInfo
    }
  
    public func notification<T: DomainEvent>(event: T) -> NSNotification {
        return NSNotification(name: T.eventName, object: nil, userInfo: event.userInfo())
    }
  

It's weird that I have to write `notification<T: DomainEvent>(event: T)` to access static properties of the event type. `notification(event: DomainEvent)` doesn't work as of Swift 1.1., because "Accessing members of protocol type value 'DomainEvent.Type' is unimplemented".

In a similar vein, a method like this doesn't satisfy the generics requirement:

    #!swift
    func foo(event: DomainEvent) {
        let notification = notification(event)
        // ...
    }

But this does:

    #!swift
    func foo<T: DomainEvent>(event: T) {
        let notification = notification(event)
        // ...
    }

Mixing protocols and generics makes me wonder where the problem lies, but hey, it does work this way, although generics pose a slight performance penalty to protocols if I remember the WWDC talks correctly.

[gist]: https://gist.github.com/DivineDominion/8e74937eac051ff26c46
[event]: /posts/2015/03/event-publisher/
