---
title: Weird EXC_BAD_ACCESS in Swift 1.2 Was Due to A Compiler Bug
created_at: 2015-09-10 16:26:39 +0200
kind: worklog
tags: [ swift ]
comments: on
---

I banged my head against Xcode's wall (which is my computer screen) today, trying to find the source of an `EXC_BAD_ACCESS code=2` termination.

Turns out it's the Swift 1.2 compiler which doesn't catch a problem here.

    #!swift
    typealias Factory = (value: Int) -> Thing
    var createThingBlock: Factory = defaultCreateThingBlock

    let defaultCreateThingBlock = {
        //(value: Int) -> Thing in  // too verbose for my taste
        //value -> Thing in         // enforced by Swift 2
        value in                    // compiles in Swift 1.2 but crashes
    
        return ConcreteThing(value: value)
    }
  
The signature of the block causing trouble with the version shown above:

    defaultCreateThingBlock: (Int) -> ConcreteThing

This works when the `Thing` protocol is for reference types only in Swift 1.2 -- until it is used. Then the process crashes.

It seems `(Int) -> ConcreteThing` should not match `(Int) -> Thing`. According to the type inheritance tree this looks good enough, but it isn't good enough for Swift.

Here follows the fully <del>working</del> <ins>crashing</ins> example Cocoa app. Remember the technique to [switch free function implementations during tests][test]?

    #!swift
    protocol Thing: class {
        var x: Int { get }
    
        func foo()
    }

    class ConcreteThing: Thing {
    
        let x: Int
    
        init(value: Int) {
            x = value
        }
    
        func foo() {
            print("success! ==\(x)")
        }
    }

    func createThingWithValue(value: Int) -> Thing {
        return createThingBlock(value: value)
    }

    typealias Factory = (value: Int) -> Thing

    var createThingBlock: Factory = defaultCreateThingBlock

    let defaultCreateThingBlock = {
        //(value: Int) -> Thing in  // too verbose for my taste
        //value -> Thing in         // enforced by Swift 2
        value in                    // compiles in Swift 1.2 but crashes
    
        return ConcreteThing(value: value)
    }

    @NSApplicationMain
    class AppDelegate: NSObject, NSApplicationDelegate {

        func applicationDidFinishLaunching(aNotification: NSNotification) {
        
            let thing = createThingWithValue(42)
            NSLog("\(thing)")
            NSLog("\(thing.x)")    // <-- crashes here
        
            NSApplication.sharedApplication().terminate(self)
        }
    }

    
It will compile with Swift 1.2, but it won't with Swift 2:

    SIL verification failed: convert_function cannot change function ABI
      ABI-incompatible return values
      @callee_owned (Int) -> @owned ConcreteThing
      @callee_owned (Int) -> @owned Thing

Thank you for reporting this, dear compiler!

Switching back to Xcode 6 and Swift 1.2 afterwards without cleaning the build products all of a sudden reports a compiler error in Swift 1.2, too. So the compiler understands the problem but it doesn't seem to have caught it.

So if your Swift 1.2 app crashes all of a sudden, look for weird type dances.

