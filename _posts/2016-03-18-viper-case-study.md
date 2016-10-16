---
title: "VIPER by Experience: How to Set Up an iOS Project"
created_at: 2016-03-18 18:22:36 +0100
kind: worklog
tags: [ viper, sync ]
url: https://swifting.io/blog/2016/03/07/8-viper-to-be-or-not-to-be/
comments: on
---

Found a very detailed article about [how a team implemented VIPER in their iOS project](https://swifting.io/blog/2016/03/07/8-viper-to-be-or-not-to-be/) written by Michał Wojtysiak & Bartłomiej Woronin.

They discuss general project architecture, using tools to generate module files, and how passing data from one Wireframe to another works in their app.

I haven't used VIPER in an iOS app, yet, but I used the concept in the Word Counter. There, nothing's disposed after setup, so I have all Wireframes and their components setup once upon launch. **The authors use Wireframes as factories** for view controllers.


## A Note about Syncing Changes: Where to Handle Sync Events

Further down, under "How to deal with listening changes from backend?", the authors show how synchronization changes take effect.

* The `SynchronizerService` merges changes into the Core Data stack,
* the Core Data Stack sends a notification,
* the Interactor receives the notification,
* and changes are then pushed through the Presenter to the View.

This works, but I find the decision to be weird. Now the Interactor prepares data when requested _and_ pushes changes upon synchronization events.

Usually, the Presenter is created in such a way that it both presents data to the view and handles events from the view. You can split this into two objects, let's say `Presenter` and `EventHandler`. Then it becomes clear that the `EventHandler` can deal with events both from UI interaction and from syncing. It's the best fit to translate any event into a command for the Interactor so it does its job.
