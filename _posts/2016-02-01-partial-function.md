---
title: Always Write Functions to Cope with all Possible Parameter Values
created_at: 2016-02-01 15:24:12 +0100
kind: worklog
tags: [ function, oop, value-object, error-handling, api ]
comments: on
---

Matt Galagher is back writing at [Cocoa with Love][cl]. His goal is maintainability, which is the greatest of all, I think. It's easy to copy code samples together to create an app, bur it's hard to create a product you can keep alive and make better over years.

In that vein, his first article, "[Partial functions in Swift, Part&nbsp;1: Avoidance][avoid]", includes a lot of details why partial functions will hurt you. This is a great topic. Read his post for the basic set theory involved.

In a nutshell, a partial function is a function that advertises to take in more values than it actually does:

- accepting `Int` as a parameter but only dealing with positive numbers properly, like array subscripts
- everything `NSArray` and `NSDictionary` did in Objective-C where you couldn't know what was inside
- similarly, accepting `id` or `AnyObject` in the API bur requiring some protocol conformance behind the scenes

This is dangerous because you have yet another detail to remember when using a function to not make your app crash. **So a partial function seems to accept a wider amount of values than it actually does, implementation-wise.**

The corollary advise is to avoid functions that raise fatal errors when the input doesn't meet expectations. Instead prefer to limit the scope of input instead, for example accepting `UInt` only. If your algorithm requires even more strict rules, create a value type to capture that. **A function should limit the parameter types in such a way that it can deal with all possible values.**

Matt's example is a non-zero integer:

    #!swift
    struct NonZeroInt {
        let value: Int
        init?(fromInt: Int) {
            guard fromInt != 0 else { return nil }
            value = fromInt
        }
    }
    
    func divideFiveBy(x: NonZeroInt) -> Int {
        return 5 / x.value
    }

Now `divideFiveBy` doesn't need to `guard` against 0 to fail silently, or crash the program with a `precondition`. The client will have the power to supply correct values 100% of the time, making the resulting app more robust. 

This may seem like overkill at first because you create more types -- but it's making things crystal clear to everyone involved. That's a huge payoff.

[cl]: http://cocoawithlove.com
[avoid]: http://cocoawithlove.com/blog/2016/01/25/partial-functions-part-one-avoidance.html
