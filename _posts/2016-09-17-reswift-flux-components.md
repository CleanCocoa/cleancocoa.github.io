---
title: Comment On "Real World Flux Architecture on iOS"
created_at: 2016-09-17 16:11:03 +0200
kind: worklog
tags: [ reswift, flux ]
url: http://blog.benjamin-encz.de/post/real-world-flux-ios/
comments: on
---

[The Flux architecture Benjamin Encz describes for the PlanGrid iOS app][fl] is like [ReSwift](/posts/tags/reswift/) only on a per-component basis: the user interactions fire change events that a store takes care of to mutate its state. The state then becomes the input for the view again.

It's much like MVVM, whereas here the view model is the store's state.

This Flux-like approach is still a big improvement over convoluted view controllers or not thinking about data flow at all. But I wonder if the gain is so much higher compared to properly separated MVC. Who reads the store's state? Only the views? Then the state is truly a view model; but how do side-effect populate to the rest of the app, like renaming or deleting things? I'd tend towards dispatching the change event to other stores, too, which may or may not react to the event. Then the view component's state is updated independently from the database, say, using the same event dispatch mechanism.

To model the flow of truth on a component basis (which is still uni-directional: from store to view) compared to a single global app state (ReSwift) sounds like a very easy to do component redesign. You can start with anything and "fluxify" it. Then move on to the next component that's particularly convoluted. You may not even want to convert all your components to this data flow approach when some components are simple enough as they are. Symmetry of component design will suffer, of course, but if you're aware of this and conclude it's better not to convert everything, that's fine.

* * * *

With ReSwift, I interpret the global app state to be a state with real (Domain) Model objects. To make a state displayable, I derive view models from it. This came very natural to me. But Ben's posts got me thinking: could there be situations where keeping view model state around is better than keeping "real" model state?

[fl]: http://blog.benjamin-encz.de/post/real-world-flux-ios/
