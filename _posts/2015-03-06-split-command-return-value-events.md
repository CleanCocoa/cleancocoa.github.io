---
title: Split Commands and Informational Return Values Apart Using Events
created_at: 2015-03-06 21:08:24 +0100
kind: worklog
tags: [ cqs, oop, east-oriented, message, domain-event ]
vgwort: http://vg08.met.vgwort.de/na/bfeaaceefb33468ba38ec95f8c593e27
comments: on
code: swift
---

When you reason about objects in your system and their means of communication, you can differentiate between three [messaging flavors][flav] according to Mathias Verraes: imperative, interrogatory, and informational messages.

* **Imperative Messages**, or Commands, produce change by _sending_ messages;
* **Interrogatory Messages**, or Queries, _retrieve_ information; and
* **Informational Messages** _send_ messages but don't expect change; they are harder to pin down until we look at an example case.

Before we can talk about informational messages and their value, let's recap the other two and see what the difference is, because them calling for being extracted is not easy to spot.

[flav]: http://verraes.net/2015/01/messaging-flavours/

## Commands, Queries, and ... Events?

**Commands** produce effects in the system. You don't expect any return value in most cases.

If you expect a return value of a Command, you either make your interfaces worse by design, or you want to know whether the operation did succeed, which is kind of okay-ish. In Cocoa, this is prominently used in file or database access methods, combined with `NSErrorPointer` to give details upon failure.

**Queries** focus on providing information about an object. They ask for the information directly, and the response is the method's return value.

Queries should not produce side-effects. Each call should be _idempotent_: performing the same Query a hundred times in a row should not change the outcome _as an effect of calling the method._  (If the object changed in between calls due to a Command coming in asynchronously, for example, the Queries' response should of course be different.)

The [separation of Commands and Queries][cqs] is nothing new. But usually, no one talks about what Mathias Verraes calls **"informational messages"**.

What are informational messages?

Consider the following three example lines of code and their return values:

```swift
canvas.clear()       // -> Void
canvas.size()        // -> Size
canvas.save(toFile:) // -> Bool
```

`clear()` obviously is a Command. It's phrased in imperative form. `size()` is a Query. It's focused on a property, phrased as a noun, returning some value.

`save(toFile:)` is phrased like a Command but returns a value just like a Query. It's hard to give this thing a name.

Let's look at how it's used to see what it is. This is `saveCanvas(_:)` of `CanvasController`:

```swift
/// Try to save to a user-picked file until success or cancelled by user.
func saveCanvas(canvas: Canvas) {
    var savingSucceeded = false
    do {
        if let destinationFile = FilePicker.pickFile() {
            // Try to save if the user picks a file
            savingSucceeded = canvas.save(toFile: destinationFile)
        } else {
            // Abort if the user doesn't pick a file
            return
        }
    } while !savingSucceeded
}
```

Mostly, it's a command, **because the caller cares about the effect**, that is data being saved. `CanvasController` commands the `Canvas` to persist to a file.

It's also an informational message, **because the receiver proactively informs about the success.** `Canvas` tells `CanvasController` (or whichever object invokes `save(_:)`, for that matter) what has happened. It doesn't expect any action, though. That's why the return value is considered _informational_.

We can split the saving mechanism apart by making the informational part a message itself instead of a return value.

Objects usually inform proactively by sending notifications to whichever object it may concern. When they send notifications, or fire off events, they don't expect a particular reaction -- or else we would've wired them to a particular object directly and called a Command.

[cqs]: http://martinfowler.com/bliki/CommandQuerySeparation.html

## Extract Failure/Success Notifications Into Informational Messages

This is how both `CanvasController` could do the save--success dance instead using events. Imagining everything else of `Canvas` and a `CanvasController` being in place, these are the methods in questions:

```swift
extension Canvas {
    func save(toFile file: File) {
       // perform file saving of the canvas' data
   
       if (didFail) {
           EventPublisher.sharedInstance.publish(
               SavingCanvasFailed(canvasId: self.canvasId))
           return
       }

       EventPublisher.sharedInstance.publish(
           CanvasSaved(canvasId: self.canvasId))
   }
}

extension CanvasController {
    func saveCanvas(canvas: Canvas) {
        if let destinationFile = FilePicker.pickFile() {
            canvas.save(toFile: destinationFile)
        }
    }

    // Call this in init(), for example, to subscribe 
    // to events automatically.
    func subscribeToCanvasEvents() {
        EventPublisher.sharedInstance.subscribeTo(SavingCanvasFailed.self) {
            [unowned self] event in
        
            // notify user about failure and prompt 
            // for picking another file
            if shouldTryAgain {
                canvas = self.canvasCollection.canvasWithId(event.canvasId)
                self.saveCanvas(canvas)
            }
        }
    
        EventPublisher.sharedInstance.subscribeTo(CanvasSaved.self) {
            [unowned self] event in
        
            // for example display subtle success message in the 
            // corner of the screen
        }
    }
}
```

Depending on the rest of the application, of course, this code can have the following benefits:

* `CanvasController` doesn't need to hold on to a `Canvas` object to understand the message because the the event sends the affected `Canvas` along. Sending a `CanvasId` as a means to fetch it from a central place instead of sending the object itself is even more cost-effective. 
* If there was only one canvas in the app, it'd be even easier to retrieve the object. `CanvasController` could even hold on to it itself. I recommend against this as this introduces higher coupling by assuming too much about the app's overall structure. This makes your code brittle.
* Saving a `Canvas` can be delegated to a background thread so the user interface doesn't block. When the process is finished, the success event will be sent. In contrast, expecting an immediate response via return values requires a blocking operation. 
* Because `save(toFile:)` can be non-blocking, `CanvasController` will itself be able to dispatch multiple saving operations, like saving all canvases into a folder simultaneously. In contrast, if the `Canvas` blocks, the `CanvasController` becomes unresponsive, too, and so on all the way up to the UI. From a user experience point of view, it's always better to be non-blocking.
* Error details could be attached to a Domain Event. An independent `Logger` can subscribe to all failures and log them. Another object can prompt the user to report fatal problems. All without explicitly passing these errors up the call chain.

Finding opportunities to tear apart potentially failing Commands and their success/failure response via events helped me a lot the last days. It made designing asynchronous processes easier and is less messy than passing blocks along as callbacks in some cases, as I suggested in [a post about error handling][error].

To model informational messages as Domain Events, or even using simple notifications for this, is often times disregarded in favor of the Cocoa pattern to return a success/failure boolean and setting an optional `NSErrorPointer` argument.

I found using events weird at first, but I see their value more clearly now. What about you? Can you point at a place in your code where events would help make your life easier and your interface more streamlined?

[error]: http://ct.dev/posts/2015/02/functional-swift-exceptions/
