---
title: Functional Error Handling in Swift Without Exceptions
created_at: 2015-02-13 17:46:02 +0100
updated_at: 2015-02-16 14:13:00 +0100
kind: worklog
tags: [ functional-programming, swift, control-flow, exception, east-oriented ]
vgwort: http://vg08.met.vgwort.de/na/9db1b8ed855f448a8579c34919a69216
comments: on
code: swift
---


In Swift, there's no exception handling _or_ throwing. If you can't use exceptions for control flow like you would in Java, what else are you going to do if you (a) write library code which executes a failing subroutine, and (b) you find unwrapping optionals too cumbersome?

I played with the thought of keeping Swift code clean, avoiding the use of optionals, while maintaining their intent to communicate failing operations.

Recently, Colin Eberhardt [pointed out](http://www.scottlogic.com/blog/2015/01/27/swift-exception-handling.html) that not throwing exceptions clutters your code with nested unwrapping conditionals:

    #!swift
    if let a = a {
      if let b = b {
        if let c = c {
          println("\(a) - \(b) - \(c)")
        } else {
          println("Something was nil!")
        }
      } else {
        println("Something was nil!")
      }
    } else {
      println("Something was nil!")
    }

**Update (2015-02-16):** With the [recent updates to Swift](http://airspeedvelocity.net/2015/02/11/changes-to-the-swift-standard-library-in-1-2-beta-1), you can get rid of the if-let-pyramid. Instead of nesting single if-let-lines, you can combine them into one: `if let a = a, b = b, c = c`. Even with the optional `where` condition this mostly sounds like syntactic sugar. Which is good, because deep nesting makes code harder to read. It doesn't help if you need to account for the `nil` case, though. Then you still end up with a pyramid.

There seems to be an alternative we can try: use closures for callbacks.

## Replacing Optionals in Return Statements With Callbacks

Of course optionals can clutter your code:

    #!swift
    let maybeResult: JSONData? = JSONParser.parse(aJSONString)

    if let result = maybeResult {
        // ...
    }

The caller has to manage the result of the callee's functionality. It has to know about what the callee does to some extend. Caring for optionals is pretty straightforward. It could be worse. But unwrapping optionals still shows in your code. It leaves traces of complexity.

Think about [the principles of East-Oriented Code][east]: Information travels west if a function returns a value. It travels east when it sends a message. An optional return value is information traveling west, and because it's an optional, it's even more complex than regular return values. Because it is _two_ cases in one call.

<!--ct: TODO write about east-oriented code and link from here-->

Nested calls and nested `if` statements are even worse.

Let's move caring for failure part of the callee's job.

    #!swift
    JSONParser.parse(aJSONString) { data: JSONData in
        // ...
    }

Now `parse()` has to take care of handling failures. Only if everything worked right will it call its success closure. If not, the caller won't know.

This may be sufficient to perform asynchronous fetches and UI updates. The parser will log or report fatal failures if needed. If the client simply can live without this request being successful, nothing will change. You can even outsource processing lots of data into an XPC service and design the interaction asynchronously from the start.

We can even explicitly handle failures if needed:

    #!swift
    JSONParser.parse(aJSONString, success: {
        data: JSONData in
        // ...
    }, failure: {
        error: NSError in
        NSLog("An error occured while parsing: \(error.description)")
    })

My key point is this: Don't be too quick to judge Swift by its cover. Maybe you can out-think problems you've got used to solving in a particular way which Swift doesn't support. [Thinking more functional][function] is one possibility.

`NSAsynchronousFetchRequest` for example [does something similar][nsafr]: it has a completion block which won't be called if the fetch request fails. The Cocoa API uses Blocks more and more. You can, too, and maybe improve your app's control flow by accident.

[nsafr]: https://developer.apple.com/library/ios/releasenotes/General/iOS81APIDiffs/modules/CoreData.html

[east]: http://www.saturnflyer.com/blog/jim/2014/12/23/enforcing-encapsulation-with-east-oriented-code/

[function]: /posts/2015/02/functional-programming-well-factored/
