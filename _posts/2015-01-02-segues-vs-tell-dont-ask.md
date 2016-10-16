---
title: Storyboard Segues VS Tell Don't Ask Principle
created_at: 2015-01-02 16:46:48 +0100
kind: worklog
tags: [ ios, design-pattern, viper, segue ]
vgwort: http://vg08.met.vgwort.de/na/c908fac3be864a928a8caaee99a47773
comments: on
---

`UIStoryboardSegue`s are a blessing to add rudimentary navigation between view controllers in iOS projects. User interaction can trigger segues without any code. That's all good and well to move back and forth between mostly static scenes in your Storyboard.

Passing data around is not so easy, though, without making things more complicated.

Interface Builder has no means to know about data passed from the source view controller to the destination view controller. That's why segues trigger a hook on the source view controller: each segue invokes `prepareForSegue:`. That's where you'd pass data on to the destination view controller. In a master--detail-application, like the list of iOS photo albums or iTunes playlists, this can work really well. Just tell the destination view controller which data to display right before the segue is performed. The rest happens automatically.

Segues replace the delegate pattern to dismiss presented view controllers. Before segues, you invoked the delegate's callback method and handled the rest. Say you present a view controller modally. The modal view controller's delegate probably is the presenting view controller. If the user is done with the presented view, the delegate is notified and takes care of dismissing the modal view controller.

Modally presented view controllers used to send data back to their delegate. Like when a new entity's data was entered in a form, you pass back the data to the delegate. The delegate creates a new entity and dismisses the form.

To dismiss modally presented view controllers today, you can use [custom unwind segues][unw] instead. The presenting view controller has to implement a method similar to `-(IBAction)unwind:(UIStoryboardSegue *)segue`. Afterwards, other view controllers can unwind in the navigation hierarchy back to this view controller via the "Exit" proxy object in their Storyboard scenes. Or you can rely on the standard "Back" navigation item and not worry about custom unwind segues at all.

Again, unwind segues, like their counterparts, are extremely useful to navigate scenes which either present static content, content they can fetch themselves, or content which can easily be passed through during `-prepareForSegue:`.

Replacing the delegate pattern with unwind segues and passing data back to the presenting view controller via `-prepareForSegue:` works just as well.

I don't think you should do this, though.

Let's say you've got a `ListViewController` and an `EditViewController` containing a form to edit list items. You select a list item. `ListViewController` performs the segue and passes the item's data to `EditViewController` during `-prepareForSegue:`. You edit the data and hit the "Save" button. The unwind segue back to `ListViewController` is performed one way or the other, passing data back to the list via `-prepareForSegue:`, this time called on `EditViewController`.

How do you pass data around? By exposing read-write properties in both places. That's the easiest way after all.

To expose mutable properties for data exchange is a very weird idea. It can potentially be mutated from someplace else, too. Would this break things? How do you tell apart "add new item" from "update existing item"? Would you create two different mutable properties on `ListViewController`?

To me, we seem to pay for a bit of convenience upon setup with too many drawbacks on the side of our code's actual design. Instead of telling our `ListViewController` to do something, we rely upon callbacks and fiddle with state instead of commands.

[Tell, Don't Ask][tda] is a powerful principle. It helps you keep code easy to read. To decipher how the form's data reaches `ListViewController`, you may have to look at existing segues in the Storyboard, then look at how data is passed back to the presenting view controller, then look at `ListViewController` to find out how this state is interpreted. That's a lot of interpretative overhead for little gain.

Compare that many levels of indirect state mutation to the more traditional `[self.delegate editViewController:self didUpdateItem:theItem]`. It's straight-forward and easy to read. The presented view controller calls its delegate to tell it that the job's finished.

To make it look less delegate-like, it'd even work as `[self.delegate updateItem:theItem]`, but then the method signature won't scream "this is a presented view controller's delegate callback" at you anymore. This is less of a problem from `EditViewController`'s point of view than it is for understanding `ListViewController`'s responsibilities.

I'd say that a view controller shouldn't have to display its data and worry about presented view controllers's callback methods. (If you separate "add" from "edit", you'd have two already. Or four if you add cancellation callbacks instead of passing `nil` instead of an actual item on cancel.)

Instead, I prefer to opt in to [VIPER architectural pattern][viper] and it's separation of event handling, data presentation commands, view interaction, and changing the navigation history.

While updating [Calendar Paste][calpasteapp] to flawlessly work with the new iOS SDK, use modern technologies, and add actual features, I found passing data in segues a burden. For other iOS apps I developed, I adhered to VIPER from the start and found the flow easy to comprehend once I got used to the patterns. When _Calendar Paste_ is ready for the future, I think about refactoring it some more to get rid of some of the Storyboard segues again.

<%= advertise :macappdevbook %>

[unw]: http://spin.atomicobject.com/2014/10/25/ios-unwind-segues/
[tda]: https://pragprog.com/articles/tell-dont-ask 
[viper]: http://www.objc.io/issue-13/viper.html
[calpasteapp]: http://calendarpasteapp.com
