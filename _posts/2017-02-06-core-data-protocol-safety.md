---
title: "Core Data is Invasive. You Can Hide It, Or You Can Embrace It"
created_at: 2017-02-06 16:55:52 +0100
kind: worklog
tags: [ core-data, domain, protocol, swift ]
comments: on
---

Found [this nice post](https://swifting.io/blog/2017/02/05/35-structs-alternative-using-swift-protocols-to-enhance-safety-of-core-data-access) about using Swift protocols to expose read-only properties of Core Data managed objects so you don't couple your whole app to Core Data.

Using Swift protocols in Core Data `NSManagedObject`s is a great way to limit the visibility of properties and methods. In [my 1st book on Mac app development][macappdev] I talked about this, too, and this is a lot easier to handle than a custom layer of `struct`s that you have to map to `NSManagedObject` and back again. Core Data is designed to be invasive and convenient. It's not designed to be used as a simple object-relational mapper.

If you have more than just data, like a fairly complex Domain Model with rich behavior and nested sub-components, then using protocols to expose read-only data won't cut it, though. This approach is super useful to expose properties for reading when their types are built-in or part of Foundation -- in other words, when Core Data knows what to do with them. But if you create your own `Tree` type with a `[Banana]` property, this again won't help.

When you plan your app, think hard about the real use of your database/persistence layer. Core Data can do a lot of the tedious work and you can easily sync stuff over iCloud. But if you really need a custom database, don't try to force Core Data into the equation.

The age-old "pro tip" of deferring decisions about persistence frameworks is sound, but you may have a harder time adding Core Data _late_ in the project than adding it _early._ As I said, Core Data is invasive, but you can use this to your advantage if you know that the amount of coupling to Core Data is just what you need. 

If all you can think about is clean code and awesome software architecture, this will feel like betraying your own principles. It's an utterly pragmatic choice, based on the principle of "does this help us ship?"

[macappdev]: https://leanpub.com/develop-mac-apps-clean-architecture-swift/
