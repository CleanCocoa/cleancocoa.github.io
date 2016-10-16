---
title: Block-Based API vs. Delegates – a Comparison
created_at: 2016-01-29 09:57:58 +0100
kind: worklog
tags: [ closure, protocol, testing, locality, memory-management, intent ]
vgwort: http://vg01.met.vgwort.de/na/23abbf449c844521b0eb7cc987a25f29
comments: on
---

As I'm exploring the use of block based API, which means to assign closures or functions handles to properties or pass them around as parameters to other functions, I found a few benefits and drawbacks in comparison to protocol-based object interactions. Here's a breakdown of criteria for blocks and delegates.

## Locality

Using closures improves readability when *defining* the callback:

    #!swift
    hardWorkingOperation { print("done") }

The event handling is right there next to the function call. Delegates and their functionality are defined elsewhere, depending how you organize your code.

Custom observers or notification center subscribers can use a block-based API, too.

## Retain Cycles

When you define the callback where it's needed, it is easy to see which objects will be captured by the closure.

    #!swift
    pigs.whenTheyFlyHandler { [weak self] in
        // Allow me to pass away before this day comes ...
        self?.run()
    }

You can pass a `[weak self]` in to avoid circular dependencies. Or you define the closure as `@noescape` to tell clients the closure will not be kept around for very long and that memory management will not be an issue -- with `@noescape` blocks, the Swift compiler will not force you to prepend calls with `self` because it's not captured for long.

Delegate protocols, on the other hand, are said to be prone to retain cycles: when objects accidentally own each other it's harder to tell because the locality is worse. When there are two files to take a look at in order to avoid over-retaining, it's easier to cause problems.

That's why delegates per convention are stored with a weak reference.

    #!swift
    class Service {
         weak var delegate?: ServiceDelegate
    }

## Expressing Intent

"Closure" and "declarativity" seem to go hand in hand. You specify what you want to happen. A closure is the action itself. So when you read a method invocation with a trailing closure, you can see what will happen in the callback. Thanks to the high locality, closure definitions can be easy to comprehend.

Creating a class which implements a delegate protocol takes only a few lines in Swift, but is it expressive? This depends on the protocol. Are its methods cohesive or did you cobble them together just to save some typing defining multiple protocols?

When your intent is "handle every event the view controller emits", a protocol with otherwise unrelated actions may capture your intent best. It's merely a tool for you, the programmer, to organize code in different objects.

Communicating intent is an important criterion for clean code. But it's also very hard to point at violations in some cases. 

When you model the underlying domain, it's easy to create expressive objects. We talk about `Banana`, `Food`, `Shipment`, etc. there. The view layer, on the other hand, is a pure implementation detail. There you'll have `XYZData`, `ABCResponder`, and whatnot. It depends on the Cocoa framework. Better not mix them both.

So the criteria for expressivity are different for the different layers of your app. It matters to split a massive view controller into smaller objects to keep the flow of information manageable for readers of your code ("future you"). But 
you'll end up with view models, presenters, and other view-related helpers.


## Expectations About Behavior

Although closures reveal what will happen when you define them, closures don't reveal what's happening when you *use* them:

    #!swift
    func prepareTastyMeal(completion: (Meal) -> Void) {
        // ...
        completion(meal)
    }

Is the meal going to be devoured, analyzed, or printed to the console once passed along? You cannot know what clients are doing with it. This is good as it decouples both parts. This is also bad because you don't know where to look for invocations.

When I design event handlers objects, delegates, and data sources, I usually create contracts that explicitly reveal what's going on based on the name. The name matters because related concepts, named properly, make reading code easier. Most protocols you find are rather specialized: UIKit calls it `UITableDataSource`, not just `UIDataSource`, for a reason. It's clear from the name that whichever class implements this protocol, _it'll be used for a table view._

The very generic closure example above, on the other hand, doesn't reveal that much. In part, this is due to the block you pass in not having to have a specific name, unlike method definitions in a protocol.

On the upside, when such a tight conceptual coupling isn't necessary, say for button event handlers, you don't have to come up with silly names. That's a bonus. You can design independent methods that aren't coupled to clients, which may promote cleaner code.

## Avoiding Weird Bugs

Following from the last point, it's easy to mess things up with closures:

    #!swift
    viewController.startVideoHandler = NSApp.terminate
    viewController.exitHandler = startPlayback

Notice anything?

The parameter-less event handler you assign to any of these actions may do anything. It may pop up a video or terminate the app. Maybe you accidentally mixed up two handlers like I did in the example above -- and you won't notice until you run the app and hit a button. Instead of playing the video, the app quits (or crashes?). Oops!

There's just no way to discern both handles by their meta information -- both have the same signature: `() -> Void`. You can only tell them apart from their actual behavior from the point of view of your app. The behavior is nothing the compiler can inspect. It's up to you to find that out.

With a type, be it protocol or class or struct, you can assert what kind of object was assigned at least.

You as the coder also read the names of the methods you assign, of course, and can infer the behavior. But you cannot guarantee not to make a mistake when you write such code. Since the compiler won't help you here, to prevent future problems, you'd ideally want unit tests to verify the setup automatically.

Which brings us to the next point.

## Verbosity in Tests

Unit tests got our backs. Except when you cannot test something.

Catching the setup from above as a failure in unit tests is not very easy because you cannot compare block attributes. You cannot check if `exitHandler == NSApp.terminate`, for example. There's just no way for Swift to tell. 

To find out if the handler was set up properly, you have to _execute_ the block and see what happens. That's nice for testing the use of the `exitHandler` property internally, for example:

    #!swift
    func testButtonCallbackInvokesExitHandler() {
        // Given
        var didInvoke = false
        viewController.exitHandler = { didInvoke = true }
        
        // When
        viewController.exitButtonPressed() // is an @IBAction
        
        // Then
        XCTAssert(didInvoke)
    }

This doesn't help at all to see how the object under test is set up, though.

A closure property is opaque, a black box with input and output, but you cannot inspect its internals.

That entails that although it's possible to pass `NSApp.terminate` in and write less code, you will end up wrapping the call in your own object if high testability is important to you to provide test doubles:

    #!swift
    class TerminateApp { 
        init() { }
        func terminate() { 
            NSApp.terminate()
        }
    }

    class Configurator {
        // ...
        lazy var terminator: TerminateApp = TerminateApp()
    
        func setUp() {
            viewController.exitHandler = self.terminator.terminate
        }
    }
    
Then in your tests:

    #!swift
    func testWiresExitHandlerToTerminatorsTerminate() {
        class TestTerminator: TerminateApp {
             var didTerminate = false
             func terminate() { didTerminate = true }
        }
    
        let terminatorDouble = TestTerminator()
        configurator.terminator = testTerminator

        configurator.setUp()
        configurator.viewController.exitHandler?()
        
        XCTAssert(terminatorDouble.didTerminate)
    }
    
A bit inconvenient to create all these types when all you want to do is observe which object's method is passed as the event handler, isn't it?

This is how you'd create delegate/protocol based test doubles, too. Although we had to create boilerplate objects to encapsulate the calls here, it's what we did in the past anyway. Coming from the closure-based one-liner, this may seem like a lot of extra work, but in the end it's just the work you had to do without closures anyway. 

There's one other remedy left: you can [replace free functions in tests](/posts/2015/07/refactoring-legacy-free-function/) when they delegate to a global mutable block variable. That's a lot less typing, but it pollutes the global namespace:

    #!swift
    var terminateAppBlock: () -> Void = { NSApp.terminate() }
    func terminateApp() {
        terminateAppBlock()
    }
    
    class Configurator {
        func setUp() {
            viewController.exitHandler = terminateApp
        }
    }

In tests, you provide a different block implementation and test its invocation:

    #!swift
    var originalTerminateAppBlock: (() -> Void)!
    
    override func setUp() {
        super.setUp()
        originalTerminateApp = terminateAppBlock
    }
    
    override func tearDown() {
        terminateAppBlock = originalTerminateAppBlock
    }
    
    func testWiresExitHandlerToTerminateApp() {
        var didTerminate = false
        terminateAppBlock = { didTerminate = true }

        configurator.setUp()
        configurator.viewController.exitHandler?()
        
        XCTAssert(didTerminate)
    }

This is less code in production but more code in tests. Find out for yourself which way scales best for your app. I wouldn't pollute the global namespace of my _Word Counter_, for example, but for smaller apps with a handful of actions this can be okay.

Unit testing works best with objects, I'm afraid. Global block variables can be a replacement that does the trick, too.

Summing up this section, when I end up with wrapper types around Cocoa objects to verify simple API calls because there's no way to see which closure was added where, I wonder if adding an explicit protocol would have made all this a bit easier to read with literally no extra code.

## Heuristics

In general:

- Block-based event handlers are very nice to use. View controllers don't have to come with a protocol each and each handler can actually delegate to another object.
- Block-based process feedback is focusing on the result (success or error) whereas delegates are more often used to [receive in-progress feedback](http://blog.stablekernel.com/blocks-or-delegates/).

As an alternative to blocks and delegates, always keep [notifications or your own observer](https://sandofsky.com/blog/delegates-vs-observers.html) collection in mind to decouple parts further.

Deciding which kind of handler to use depends on a lot of factors. These are my current heuristics:

* I use closures as callbacks for actions, especially if they run async.
* I use closures as event handlers for simple interactions.
* I prefer protocols when the interaction is more complex than a single method invocation; I wouldn't enjoy replacing `UITableViewDataSource` with a ton of closures because the result would be less cohesive.
* I prefer protocols to report progress back during long-running processes.
* I prefer protocols when I test code. (And I test most of my code.)

Did I miss an important criterion? Do you do things differently? **Share it in the comments!**
