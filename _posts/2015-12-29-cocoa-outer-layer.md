---
title: Making Cocoa the Outermost Layer
created_at: 2015-12-29 11:38:04 +0100
kind: worklog
tags: [ software-architecture ]
comments: on
image: 201512291347_app.png
---

Chris Eidhof of objc.io put some [experimental code on GitHub](https://github.com/chriseidhof/shoes-swift/blob/master/app.swift) where quite a few Cocoa view components are contained by classes he owns himself.

The AppDelegate is super instructive:

    #!swift
    class MyAppDelegate: NSObject, NSApplicationDelegate {
        let window = NSWindow()
        var didFinishLaunching: NSWindow -> () = { _ in () }
        func applicationDidFinishLaunching(aNotification: NSNotification) {
            didFinishLaunching(window)
        }
    }

Your Cocoa app needs an `NSApplicationDelegate`. But it doesn't _have_ to do anything except route events to the proper collaborators. Still it's the first object we usually put logic in only to (hopefully) refactor it out later.

Think about the plethora of `NSApplicationDelegate` methods out there. Sprinkle in some URL Scheme responder actions. What if the method bodies were mere one-liners? Then your AppDelegate becomes very easy to test and maintain. The flow of information and especially commands is clear: away from the delegate to specialized event handlers.

Now Chris seems to favor free-floating functions for factories. That makes sense. But I wouldn't put event handlers into these. He doesn't either; that's what `ButtonDelegate` is good for:

    #!swift
    class ButtonDelegate: NSObject {
        var callback: () -> ()
        init(_ callback: () -> ()) {
            self.callback = callback
        }
        @objc func buttonClicked() {
            callback()
        }
    }

The final weird, and interesting part, is the actual setup of the app:
    
    #!swift
    app("My app") { theApp in
        let text = Array(count: 3, repeatedValue: "Hello, world").joinWithSeparator("\n")
        let tv = textView(text, editable: true)
        let theButton = button("Hello") { tv.text += "\nHello!"}
        let buttons = stack([theButton, button("Exit", onClick: theApp.exit)], orientation: .Horizontal)
        return stack([label("Add some text"), tv, buttons])
    }

This code results in this app:

{% include figure.md src="/assets/blog/2015/201512291347_app.png" alt="app screenshot" caption="Resulting app" %}

Not bad for 5 lines of actual implementation code.
