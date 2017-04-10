---
title: Express Either–Or-Relationships as Enums
created_at: 2017-02-11 17:41:21 +0100
kind: worklog
tags: [ swift, enum, encapsulation ]
comments: on
---

If you want to encapsulate the notion of "either A or B" (also called "lifting" in functional parlance), an enum type in Swift is the best fit:

    #!swift
    enum EitherOr {
        case A, B
    }

You can use associated values to wrap types with enums, too:
    
    #!swift
    enum Fruit {
        case banana(Banana)
        case apple(Apple)
    }

These things seem to be expressible through a common ancestor type or protocol. But bananas and apples can be modeled in totally different manners (apart from sharing nutritional value, for example).

A real-life example is the result of a network request. Instead of a classic Cocoa callback, express the strict either-or through a type:

    #!swift
    /// Only one is ever supposed to be present, but you never
    /// know at the call site.
    typealias ResultCallback<T> = (result: T?, error: Error?) -> Void
    
    /// Only one can ever be used, never both.
    enum Result<T> {
        case success(T)
        case error(Error)
    }
    typealias ResultCallback2<T> = (Result<T>) -> Void
