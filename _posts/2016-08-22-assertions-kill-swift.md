---
title: Assertions in Swift Kill Your Code
created_at: 2016-08-22 16:41:07 +0200
kind: worklog
tags: [ swift, defensive-programming ]
vgwort: http://vg01.met.vgwort.de/na/2e858a6a9b254808b98296843f838d85
comments: on
---

`Swift.assert` bit me, and it bit pretty hard.

What do you expect to happen in this code:

    #!swift
    func sendAction(application application: NSApplication = NSApp, sender: AnyObject? = nil) {

        assert(application.sendAction(self.selector, to: nil, from: sender))
    }

... compared to this code:
    
    #!swift
    func sendAction(application application: NSApplication = NSApp, sender: AnyObject? = nil) {

        let success = application.sendAction(self.selector, to: nil, from: sender)
        assert(success)
    }

If your answer is "only the second variant will actually execute `sendAction` in release builds," then you're way smarter than I.

_Until now_ I assumed that `Swift.assert` behaves similar to assertion macros in Objective-C where the code is passed through in release builds. I never cared to read the documentation:

> In -O builds (the default for Xcode's Release configuration), condition is not evaluated, and there are no effects.

This was the source of a maddening bug in TableFlip for quite a while which only occured in release builds as I found out today. Huh. So much for defensive programming in one-liners.

I'm a bit shocked that the autoclosure parameter isn't even evaluated in release builds. But now I know. I bet I'll never forget this. Don't ask how long it took to find this. (Hint: 3 hours in total, maybe.)
