---
title: Return Types can Capture Async Processes and Failures
created_at: 2015-04-16 12:14:39 +0200
kind: worklog
tags: [ functional-programming, swift, control-flow, exception, concurrency, error-handling ]
shared_image: "blog/201503012144_guard-t.jpg"
vgwort: http://vg08.met.vgwort.de/na/c15ea3d4e7a64f918b19fac26a051961
comments: on
---


I've written about [using the Result enum to return values or errors][respost] already. `Result` guards against errors in the process. There's also a way to guard against latency, often called "Futures". There exist well-crafted solutions in Swift already.

Lately, I've been [watching interesting talks][talks]. Eric Meijer's talks taught me that honest functions communicate what they do in their return types. Swift doesn't have honest functions like Haskell does: side effects can always occur. But Swift is more honest than Objective-C: instead of throwing exceptions, you have to use something like the `Result` enum. To indicate a value may be nil, you have to wrap it in optionals. That's a huge step forward.

Then I found GitHub repositories for working implementations of [`Result`][result] (boxing values with `[Box][box]`) and [`Deferred`][deferred], which is an implementation of Futures.

`Deferred` return values return immediately, but they are just containers. They contain no real value until the process finishes and fills something in. As a client, you act on the real value contents using blocks:

    #!swift
    let deferredResult: Deferred<Something> = performOperation()

    deferredResult.upon { result: Something in
        println("got \(result)")
    }

You can even use common monadic data transformation functions like `map` and `bind` to chain deferred results together:

    #!swift
    // Producer fills in a String asynchronously
    func readString() -> Deferred<String> {
            let deferredResult = Deferred<String>()
            
            // dispatch_async to fill deferredResult
            dispatch_async(dispatch_get_main_queue(), {
                let result = computeComplicatedResult()
                deferred.fill(result)
            })
            
            return deferredResult
    }

    // Consumer transforms the String to Int ...
    let deferredInt: Deferred<Int?> = readString().map { $0.toInt() }
    
    // ... and uses the resulting number when it's ready.
    deferredInt.upon { println "next int: \($0 + 1)" }

If an async operation may fail, you can use `Result` and `Deferred` together. 

A function guarding against latency, errors, and optionals, which indicate successful operations without a return value, may have a rather long signature:

    #!swift
    func foo() -> Deferred<Result<Optional<Int>>>
    func foo() -> Deferred<Result<Int?>>          // equivalent

You will now with absolute certainty what to expect from this function (except side-effects). The client code will not be blocked and you can handle errors in a convenient way.

Sounds like too much hassle? Well, think about it this way: the nested generics-based return type is a mouthful, but you don't have to act on any of it until you know you want to. Until then, you can pass it around. Working with the `Result` enum is more complicated on the surface because you have to unwrap the value in case of success -- but once you do, you can simply use the real value and its type to communicate with other objects.

Imagine the implications for writing highly responsive user interfaces: no more blocking the main thread while you wait for data! The same holds true for waiting for server responses, no matter if you talk to a remote web server or to a local XPC service.

These little gems will make your life easier. If you use them, you have to think differently in your code: you have to _design_ your application to work asynchronously or to handle error cases gracefully.

At least now you will know what to do when.


[talks]: /posts/2015/04/11/functional-swift-talks/
[respost]: /posts/2015/02/beyond-guard-clauses/
[result]: https://github.com/bignerdranch/Result
[deferred]: https://github.com/bignerdranch/Deferred
[box]: https://github.com/robrix/Box
