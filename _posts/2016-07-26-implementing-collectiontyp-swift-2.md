---
title: Implementing CollectionType in Swift 2.x
created_at: 2016-07-26 17:19:59 +0200
kind: worklog
tags: [ swift, protocol, pop ]
comments: on
---

I ran into trouble with custom collections the other day and wondered how to reliably find out what the bare minimum for a custom `CollectionType` is. 

The compiler will only help a bit:

    #!swift
    struct Foo: CollectionType {}

It's conforming to neither `CollectionType` nor `SequenceType`. Now I know `SequenceType` requires the `generate()` method -- but don't let this fool you. `CollectionType` will provide that for you if you meet its requirements.

You'll need:

* `startIndex`,
* `endIndex`, and
* `subscript`.

The indexes have to be of the same kind; the parameter for `subscript`, too. Here's a very simple custom collection that raises an exception when you reach a point past the end, just like `Array` does:

    #!swift
    struct Foo: CollectionType {

        var startIndex: Int { return 0 }
        var endIndex: Int { return 10 }

        subscript (index: Int) -> String {

            precondition(index < endIndex)

            return ".\(index)."
        }
    }

    print(Foo().map { "x\($0)x" })
    // ["x.0.x", "x.1.x", "x.2.x", "x.3.x", "x.4.x", "x.5.x", 
    //  "x.6.x", "x.7.x", "x.8.x", "x.9.x"]

You can leave the `precondition` out for stuff like mapping and return an optional `String?` instead. Works well, too, but changes you collection's type, of course:

    #!swift
    
    struct Bar: CollectionType {

        var startIndex: Int { return 0 }
        var endIndex: Int { return 10 }

        subscript (index: Int) -> String? {

            guard index < endIndex else { return nil }

            return ".\(index)."
        }
    }

    print(Bar().flatMap { $0 }.map { "_\($0)_" })
    // ["_.0._", "_.1._", "_.2._", "_.3._", "_.4._", "_.5._", 
    //  "_.6._", "_.7._", "_.8._", "_.9._"]
