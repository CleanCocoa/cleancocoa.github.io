---
title: How to Save File Changes Using ReSwift Actions
created_at: 2017-01-17 08:07:40 +0100
kind: worklog
tags: [ reswift, file, io ]
image: 20170117080653_filesaving.png
vgwort: http://vg01.met.vgwort.de/na/7646672b8fcd4550a7d6ec368b126534
comments: on
---

Let's say you write a plain text editor that can work with multiple text files at a time. As a backbone for processing information, you use ReSwift to model the data flow in a clean fashion. The user has 10 files open, changes the text in 1 file's tab. The user leaves the text field and your awesome autosaving is triggered. 

How will saving work? How do you get the string from the text view of your user interface into a file -- using ReSwift actions?

I came up with something that seems roundabout at first but makes sense the longer I think about it. 

**Update 2017-02-23:** I wrote up some code details in [another post about `PendingFileChanges`.](/posts/2017/02/reswift-enqueue-file-changes/)

The problem in planning this: Dispatching actions in ReSwift should not trigger side-effects immediately. It's a mechanism to change the global app state, not a notification mechanism. So there may be no direct observers for the autosave event.

I always ask myself: who might want to know about the event? Now since it's not allowed to be notified directly, how can I model this as a part of the app's state without getting in the way of the rest? (It may be a code smell if every event has its own app state property equivalent and you misuse the store as an event recorder.)

In the case of saving files, there may be something like a `FileSavingService`. It wants to know when it should do something. Autosaving should happen every now and then, but not necessarily _immediately_ after the user enters a few characters and then stops. It can happen with a delay. (But it doesn't need to. Immediate is just as fine.) So a _queue_ of file changes makes sense. The app enqueues file changes and the `FileSavingService` processes the queue.

In short:

{% include figure.md src="/assets/blog/2017/20170117080653_filesaving.png" alt="sketch of the data flow" caption="The data flow, visualized" %}

* Dispatch a `EndingEditing` action with a reference to the current `file` and the file's new `contents`.
* Reduce `EndingEditing` actions to enqueue the tuple of `(file, contents)` (or a dedicated value type, of course) in a substate of the overall app state.
* Make the `FileSavingService` a `ReSwift.StoreSubscriber` and process the queue -- on a background thread.

I find this to be roundabout because traditionally, you want to wire ending editing to invoke a `saveCurrentFile` method or something. User interaction directly maps to persistence. With this approach, you introduce buffers. User interactions can happen at will and emit "please save these contents sometime" events without directly doing anything. No user interface action handler knows what is really going to happen. All they do is dispatch events. This thinking comes easier to me now than when I started with ReSwift, but it's still surprising how this "Redux"-approach influences modeling processes in my apps.

