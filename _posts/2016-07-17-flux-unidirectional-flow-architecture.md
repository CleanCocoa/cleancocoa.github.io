---
title: How PlanGrid uses a Flux-Inspired Architecture to Simplify State Management
created_at: 2016-07-17 10:40:05 +0200
kind: worklog
tags: [ state, flow, 1df, software-architecture, reswift, flux, redux ]
url: http://blog.benjamin-encz.de/post/real-world-flux-ios/
comments: on
---

Benjamin Encz [published an article][ben] about the architecture of PlanGrid.

They moved to a [Flux][]-inspired "unidirectional flow" approach where state changes are performed in a `Store` (in the backend, if you will) and updates pushed to the user interface (the frontend). No ad-hoc view updates, no shortcuts. User events are emitted, handled by the store, and state updates affect the view. That's it. So it's always obvious how the user interface got into the state it's in: all the details are plainly visible in the current state object.

Ben's post is very deep and detailed. You'll learn a ton from it:

* architectural discussion: why "unidirection flow", and why Flux?
* setup of actions/events and the store type
* basic state--view bindings via Reactive Cocoa
* how testing stores (and views) is straightforward with this setup

I can't wait for the next post in this series. Most of this easily applies to [ReSwift](/posts/tags/reswift/), too, of course. And guess who just got inspired to refactor [TableFlip](http://tableflipapp.com/) so the model and view become even more loosely coupled?


[ben]: http://blog.benjamin-encz.de/post/real-world-flux-ios/
[flux]: https://github.com/facebook/flux
