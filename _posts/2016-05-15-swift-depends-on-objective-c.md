---
title: Swift Depends on Objective-C
created_at: 2016-05-15 11:22:30 +0200
kind: worklog
tags: [ swift ]
comments: on
---

Brent Simmons [brings up](http://inessential.com/2016/05/14/the_tension_of_swift) valid concerns about the current state of Swift: you cannot write any software using Cocoa (AppKit for Mac or UIKit for iOS) 
 with Swift without somehow interfacing with the Objective-C runtime. That's kind of weird , don't you think?

When I write apps with Swift, I always feel most comfortable in the model layer. And everything else thats just a `struct` or `class Foo` without inheriting from a framework. It's 100% mine. I love that about using Swift.

I guess that's part of why Russ Bishop [wrote a few types](http://www.russbishop.net/packing-bytes-in-swift) to encode `Float32` without any external dependencies: it's possible, but it's foundational work, so to speak. The exercise is interesting, but usually you'd just use `NSData`, for example.

So when you want to display something on screen, there's Apple's frameworks waiting for you to be imported. Depending on `NSObject` and the dynamic dispatched stuff Objective-C borrowed from Smalltalk -- things you find in scripting languages like Ruby, too. But which are completely absent in pure Swift. 

The 20 years of design that went into Carbon, Cocoa and Objective-C, as [Daniel Jalkut](http://indiestack.com/2016/05/brents-swift-tension/) points out, adhere to a different philosophy than Swift. But Daniel and Brent's point isn't that Swift will do away with all that and we should be afraid. It's that Swift promises more than it can keep at the moment. It's not independent. At least not if you're using it outside its limited Open Source'd scope. When you write apps for Apple devices, Swift is bragging to do all the things while behind the scenes good ol' Objective-C is pulling most of the strings related to actual user interactions.

From time to time, I get my fix of beautifully relaxing and flexible programming with Ruby. Because I cannot make use of pure Smalltalk. Swift doesn't speak the same language. When Ruby is the charismatic bard in your role-playing party, Swift is the elven strategist. One charms the ladies while the other lays out the plans. One is cool while the other strict. 
