---
title: Debugging NSBeep Error Sound in NSResponder Method Calls
created_at: 2016-11-12 08:58:30 +0100
kind: worklog
tags: [ nsresponder, debugging ]
vgwort: http://vg01.met.vgwort.de/na/2507806dd0d54ff0b420b8700be57cdd
comments: on
---

I utilize `NSResponder` actions in TableFlip to move the selection around. Naturally, neither a standard `NSTableView` nor a custom `NSResponder` implement default behavior for (most of) the methods I need to support arrow key movement, tab movement, and the usual shortcuts, like Cmd-Left to jump to the first cell in the current row. Or Alt + arrow keys to insert a row or column next to the selection. So I implemented this in a custom `NSResponder` subclass.

Curiously, the app beeped when I hit Alt-Up or Alt-Down, the actions to insert a row above or below the current selection. I implemented `moveToBeginningOfParagraph(_:)` to consume these events but still the system "input error" sound was audible while the key combo itself performed the action just fine.

How do you find out the source of problems like this? Well, this is how I dug in.

My setup: 

* The custom `NSTableView` subclass overrides `keyDown(_:)` with a call to `nextResponder?.interpretKeyEvents(_:)` -- but not delegating to `super`, thus consuming the event, to disable standard `NSTableView` shortcuts.
* Note that `keyUp(_:)` would in general produce a similar effect but probably beep for every shortcut; `keyDown(_:)` has a special meaning in all this, so make sure to override this method.
* The next responder implements `moveToBeginningOfParagraph(_:)`. This method is called just fine thanks to `interpretKeyEvents(_:)` being called. This method maps common shortcuts to `NSResponder` method calls.

If you have a custom `NSView` subclass, you can call `self.interpretKeyEvents(_:)`, too, instead of pushing it to the next responder; it's just that I want to bypass the table view in this particular case. `NSTableView.interpretKeyEvents` results in unwanted behavior.

This is where the beep happened even though the Alt-Up/Alt-Down `NSResponder` methods were present. The input error sound usually plays when you don't implement a shortcut anywhere in the responder chain.

According to lore and the docs, `performKeyEquivalent(with:) -> Bool` should be useful: you return `true` to indicate that the receiver took care of the event selector passed to it. In theory, this should prevent the beep sound even when you don't implement the `NSResponder` method -- but that didn't work at all for me.

{% include figure.md src="/assets/blog/2016/201611120901_breakpoint.png" alt="screenshot of Xcode breakpoint" caption="Symbolic breakpoint in Xcode" %}

These are the steps I found useful to hunt for the source:

1. Add a _symbolic breakpoint_ in Xcode for symbol "NSBeep" in module "AppKit". When you hit enter or run the program for the first time, the breakpoint should obtain a child item; that indicates the symbol is recognized. Run the program, hit the key combo, and Xcode should stop execution. Look at the stack trace to see from where the key interpretation reaches nirvana. -- I found out that shortcut execution indeed reaches `interpretKeyEvents(_:)` but seemingly bypasses the rest of the responder chain which should handle the expected `moveToBeginningOfParagraph(_:)` call. (At least when it beeps. Since the action is performed by the program, I know the method is invoked.)
2. Add `doCommandBySelector(_:)` in your target responder. Insert a manual breakpoint. Look at the value of the `selector` parameter for every call.

The first step revealed that some methods were never called _when the beep happened._ The second step told me which selector was actually invoked for a single hit of Alt-Up:

1. `moveBackward(_:)`
2. `moveToBeginningOfParagraph(_:)`

Alt-Down produces a similar combo:

1. `moveForward(_:)`
2. `moveToEndOfParagraph(_:)`

Aha!

Once I added an empty implementation of `moveBackward(_:)` and `moveForward(_:)`, the beeps went away.

`NSResponder` debugging isn't much fun most of the time because of all the implicit behavior. Then again, you get a lot of useful stuff for free. Sheesh. At least now I know that `doCommandBySelector(_:)` is a great opportunity to see what's done behind the scenes because it's a bottleneck every call has to pass.
