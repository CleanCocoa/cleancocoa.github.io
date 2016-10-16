---
title: Separating State from State Changes
created_at: 2016-01-15 16:09:20 +0100
kind: worklog
tags: [ closure, oop, srp, settings, state, change ]
vgwort: http://vg01.met.vgwort.de/na/7de438f2bb2d4ddbaadb4cbf3fae14ac
comments: on
---

Reflecting on a recent change of the Word Counter's file monitoring module, I think I re-discovered a commonly advised pattern in my code: to separate state from state changes.

There's an object that knows how to handle files based on extension: plain text files's words are counted differently than Word or Scrivener files. Call it `Registry`. Previously, this was set up once and didn't change. Now I wanted to make this configurable so users can add custom plain text extensions. This means changing that object's state.

Storing these settings in `NSUserDefaults` makes a lot of sense: there's a default set of values and it should be possible to change them. Changes should persist between launches.

I made it so the set of word count strategies doesn't change automatically. It's still configured at launch but can be reset and re-configured later. It doesn't change magically when the user defaults change, either. In fact, I now find it shouldn't know how to change itself at all.

Instead of changing on its own, there's a configurator object that takes care of this:

    #!swift
    class Configurator {
        
        var plainTextExtensions: [String] {
            get { /* return user defaults object */ }
            set { 
                /* set user defaults */
                resetRegistry(plainTextExtensions: newValue)
            }
        }
        
        let registry: Registry
    
        init(registry: WordCountStrategyRegistry) {
            
            self.registry = registry
            resetRegistry(plainTextExtensions: self.plainTextExtensions)
        }
            
        private func resetRegistry(plainTextExtensions plainTextExtensions: [String]) {
        
            registry.resetKnownStrategies()
            registry.registerWordCountStrategy(txtFileWordCountStrategy, forPathExtensions: plainTextExtensions)
            registry.registerWordCountStrategy(rtfFileWordCountStrategy, forPathExtensions: ["doc", "rtf", "rtfd"])
            registry.registerWordCountStrategy(docxFileWordCountStrategy, forPathExtension: "docx")
        }
        
        // ...
    }
    
It wraps persistence of the settings to `NSUserDefaults`. It encapsulates the change (and retrieval) of the underlying data. It also takes care of keeping the user defaults and the `Registry` in sync. It doesn't use CocoaBindings, so if the user decides to use the `defaults` command line tool to change the setting while the app is running, say, then the `Configurator` will not notice. That's okay with me.

So the true state the app deals with is in the result from the `Configurator`, as opposed to the content or state of the user defaults.

So what's the big deal?

Since the `Configurator` changes the data and the app's state in lock-step, I can wire event handlers to this configurator. With Objective-C, property setters would expose `-setPlainTextExtensions:` automatically; with Swift we have to create our own if we need a method handle. I like that, I have to admit, because I prefer to pick intention-revealing method names anyway:

    #!swift
    extension Configurator {
        func configureRegistry(extensions: [String]) {
            self.plainTextExtensions = extensions
        }
    }


This method opens up the opportunity to use the configurator for any `([String]) -> Void` event handler. I don't have to pass the configurator instance around, making it impossible to change the `plainTextExtensions` or any other property directly. When I pass a method handle to a function, all that's possible from the destination is to execute the closure.

Okay, this particular configurator would work exactly the same if client code directly mutated the `plainTextExtensions` property. But if an object has more than a single thing to change, passing method handles around restricts access just as much as a protocol.

    #!swift
    let registry = Registry()
    let configurator = Configurator(registry: registry)
    someViewController.changePlainTextExtensionsHandler = configurator.configureRegistry

The separation of state and state transitions into two places helps ...

- write separate tests for state change and state representation; 
- know where to look for up-to-date information in production code, no matter what has just happened, namely the `Registry` instance;
- know how to change the state without worries.

To restrict knowledge about what an object can do, we usually had to use specialized protocols. Passing an object around as an instance of a protocol hides all implementation details. That's the way it used to work.

With closures, it's not necessary to specify protocols for all your app's different needs. You don't have to come up with clever names and deal with tons of types.

This flexibility comes at the cost of not knowing on the receiver's side what'll happen if the closure is executed. Take the common `successCallback: () -> Void` parameter, for example. The caller could have passed `NSApp.terminate` along. Is this expected behavior? Do you want that kind of anything goes flexibility? The closure might be _anything_ conforming to the expected signature.

In the end, I'm happy this is a viable option to connect components, [especially event handlers](/posts/2016/01/event-handler-closure-object/). Because most event handling protocols are just a collection of callbacks for the view controller. But I have a nagging feeling this pattern isn't well-suited for general recommendation precisely because of the reason stated above: when you code that accepts a closure, you lose a lot of information about the handler's type and origin, which makes figuring out what'll happen harder.

With time and practice, I bet I'm going to find out where this should not be used at all. At the moment I continue with caution and see what goes wrong.
