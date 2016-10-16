---
title: Custom Enumerated Sequences in Just 1 Line of Code
created_at: 2016-05-23 09:22:46 +0200
kind: worklog
tags: [ swift, functional-programming, sequence ]
comments: on
---

In ["Creating a Cheap Protocol-Oriented Copy of SequenceType (with a Twist!)"][p] I created my own `indexedEnumerate()` to return a special kind of enumeration: instead of a tuple of `(Int, Element)`, I wanted to have a tuple of `(Index, Element)`, where `Index` was a UInt starting at 1. A custom `CollectionType` can have a different type of index than `Int`, so I was quite disappointed that the default `enumerate()` simply returns a sequence with a counter for the elements at first.

Thinking about the meaning of "to enumerate" a bit more, I understand why that is so. Still I need something different that an enumerated sequence.

Now [Rob Napier](http://robnapier.net/) kindly got in touch via email and told me about some cons of my approach I wasn't quite aware of. Thanks for that!

On top of that, he showed me a version which was only 1 line of lazily evaluated code that could replace my `IndexedSequence` type:

    #!swift
    let indexedValues = zip(collection.indices, collection)

A `CollectionType` exposes a property called `indices` which is a range over `(startIndex...endIndex)`. Zipping that range with the collection itself produces a sequence of tuples of type `(Index, Element)`. Just what I needed. 1 line of code. This was so concise that I now have found a couple more places where zipping did the job better than my naive approach.

Thanks a ton, Rob.

[p]: /posts/2016/03/custom-enumerated-sequence/
