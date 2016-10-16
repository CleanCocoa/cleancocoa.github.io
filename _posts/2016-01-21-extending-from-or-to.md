---
title: "Extending Types with a Conversion Factory: Where Should You Put It?"
created_at: 2016-01-21 16:43:34 +0100
kind: worklog
tags: [ core-data, module, factory, extension ]
comments: on
preview: fulltext
---

In a post about the [Swift reflection API](http://appventure.me/2015/10/24/swift-reflection-api-what-you-can-do/#sec-2-3), I found the conversion code from a `Bookmark` struct to a Core Data object interesting. Let's call the latter `ManagedBookmark`.

What puzzles me: which object should be responsible for the adaptation from one to the other?

1. `Bookmark` converts itself to a `NSManagedObject` using a `NSManagedObjectContext` you also have to pass in. (That's what [Benedikt](https://twitter.com/terhechte) did in his post.)
2. `ManagedBookmark` provides an initializer or other factory taking in a `Bookmark` and, as always with managed objects, a `NSManagedObjectContext`.

Option 2 is pretty clear. Using `Bookmark`s public interface, the managed object will set its attributes. This requires a `NSManagedObject` subclass wheras option 1 can work with a dictionary of values.

My main worry is that in option 1 `Bookmark`, a value type of the application, has to have knowledge about Core Data. Usually we'd fare better if we separate Core Data from our own model. 

With extensions in Swift (and categories in Objective-C) we have the power to add new behavior to existing types in other locations. Does this help?

Extending a type in a different file hides that detail at first sight -- but the extension, when it's not `private`, is available in the rest of the code base, too. It just appears as if you put the code in the type definition itself.

Now if `Bookmark` is included in a "Bookmarking" module you import, an extension can add behavior without mixing Core Data visibly into the model layer. The Bookmarking module will not get altered when you extend one of its types in the client code.

I like organizing Swift code in modules because the boundaries are very strict. You cannot cheat yourself into mixing responsibilities and blurring boundaries that easily.

