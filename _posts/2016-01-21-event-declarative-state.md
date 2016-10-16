---
title: Events as Declarative State
created_at: 2016-01-21 21:15:51 +0100
kind: worklog
tags: [ state, event, flow, reswift, 1df ]
url: https://realm.io/news/benji-encz-unidirectional-data-flow-swift/
comments: on
image: 201601212137_encz-flow.png
preview: fulltext
---

I think I found something better [than closures to handle events][p]. Instead of dispatching closures that are themselves the changes that should be performed, simply _declare_ the change.

    #!swift
    enum Action {
        case ResizeBanana(BananaId, Size)
    }

Now you need some kind of action handler, which Benjamin Encz calls "reducer" in his presentation ["Unidirectional Data Flow in Swift"][en]. Think of a list of reducers as a [Chain of Responsibility](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern) only without the final consummation of the action. It is passed throughout the whole responder chain. Why reducer? Because they operate similar to the [reduce function](https://www.weheartswift.com/higher-order-functions-map-filter-reduce-and-more/). The end result will be the combined result of all reducer's mutations.

This is cool, because no component needs to know what such an action entails -- none except the appropriate reducers.

If closures were great because where you type them you can see _what is going on_ (more like "how" in my opinion), a simple action enum is even simpler, telling _what should happen._

Works well with CQRS, I imagine. Will have to write a demo app using CQRS sometime to validate.

The current project is called [ReSwift](http://reswift.github.io/ReSwift/master/) and code is available [on GitHub](https://github.com/ReSwift/ReSwift).

[p]: /posts/2016/01/event-handler-closure-object/
[en]: https://realm.io/news/benji-encz-unidirectional-data-flow-swift/
