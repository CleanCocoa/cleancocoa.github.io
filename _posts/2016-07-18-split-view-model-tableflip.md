---
title: "I'll Split Up the Monolithic View Model in TableFlip. Here's What I Expect to Happen"
created_at: 2016-07-18 17:20:02 +0200
kind: worklog
tags: [ reswift, 1df, refactoring, presenter, mvvm ]
comments: on
---

The upcoming changes in TableFlip's user experience made me ponder how I structured the app's modules. The [bottleneck](/posts/2016/07/decoupling-facade-oop/) I created in my app is simple: there's one presenter which transforms the model into a view model; the presenter defines a protocol for the view it expects and passes the view model to the object that satisfies this contract.

A single view model for everything comes with quite a few downsides. In hindsight, it doesn't make much sense that updates to the cell selection affect the `TablePresenter` which updates the view with a `TableViewModel`. It was convenient to have these two together in the beginning. Now this design decision doesn't serve me well anymore and I see the inherent flaws more clearly.

In effect, there should be 3 view models for a TableFlip document window when I'm done with the next step:

1. Document: which table is selected? How many tables are in the document?
2. Table: which cells should be displayed in the window?
3. Selection: where's the selection rectangle located?

Part of the bottleneck the `TableWindowController`. It acts as a Facade to the whole view module by satisfying the contract of the presenter. Upon updates it actually recognizes two different parts of the model at the moment: it does take care of updating the `TabBarController` with document info; and the rest is delegated to the `TableViewController`. 

Pulling selection management out of the `TableViewController` into another object will help encapsulate handling arrow keys, for example. That's a different kind of event than typing text and changing table contents. 

I imagine this re-structuring at the seam of application logic and user interface will help improve performance, too. With 3 different view model types I can make atomic view updates with less hassle.

At the moment, TableFlip still runs my initial (and naive) implementation: moving the cell selector with arrow keys triggers an update in the app state. The updated state is transformed into a view model by the presenter and then pushed to the view where a full update (!) is issued. Not very performant. "Diffing" for changes in the table model is useful to make this even faster. Updating the selectors independently from the table goes into the right direction. 

From that separation will follow even more changes that'll improve the code because I can now _think_ and design in different terms. Each document window will then be composed of 3 parts, each satisfying another contract to show information to the user. Instead of a monolithic view, there'll be 3 large sub-components.

I'm going to add some [basic reactive bindings](/posts/2014/12/mvvm-in-swift/) or even employ ReactiveCocoa for the table to make user interface updates more performant. I have no clue which way is the best to create bindings for a 2-dimensional table of varying size, yet. Let's figure this out when it's time.
