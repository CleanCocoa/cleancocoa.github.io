---
title: Test doubles for Core Data managed objects might not work as expected
created_at: 2015-06-05 12:17:11 +0200
kind: worklog
tags: [ core-data, swift, objective-c, testing, stub ]
vgwort: http://vg08.met.vgwort.de/na/448d094fb4654da38f5c88f8c2eac343
comments: on
---


You don't have to learn anything new if you work with Core Data in Swift. It's a pain to use `NSManagedObject` directly, so you better work with custom subclasses all the time. Unboxing optional `NSNumber`s is just too cumbersome.

There are caveats you should be aware of, though: Swift's static typing makes it a bit harder to write test doubles for managed objects.

A particular test suite in my current project tests a repository to fetch objects according to a certain state of a subclass of `NSManagedObject`. I don't want to deal with the real thing in tests, of course, but I need to provide a canned value for the tests to make assertions.

That calls for test stubs -- only there can be problems when you need more than one of them if you're not creating them from within Core Data contexts. I assume you want to create and pass the test doubles like usual objects, not like Core Data requires. 

This may become tricky. Let's have a look at both Objective-C and Swift.

## Trivial Test Doubles in Objective-C

In production, you create objects using `NSEntityDescription`. This is not necessary in tests. In Objective-C, we would've created another class with a similar interface and forced it into the receiver:

    // (Production Target)
    @interface ManagedThing: NSManagedObject
    // ... properties
    - (NSInteger)computeSomethingSpecial;
    @end

    // (Test Target)
    @interface TestThing: NSObject // no inheritance
    // ... same properties
    @property (nonatomic, assign) NSInteger testComputationResult;
    - (NSInteger)computeSomethingSpecial;
    @end

    @implementation TestThing
    - (NSInteger)computeSomethingSpecial {
      return self.testComputationResult;
    }

With this in place, you're now able to pass instances of `TestThing` in place of real `ManagedThing`s:

    - (void)testDoubling_WithCompuationResultOf42_Returns84() {
      TestThing *thingStub = [[TestThing alloc] init];
      thingStub.testComputationResult = 42;

      NSUInteger result = [objectUnderTest doubleResult:(id)thingStub];

      XCTAssertEqual(result, 84);
    }

That's how easy it is to write test doubles in Objective-C. As long as the interfaces are similar for the scope of the method you're testing, you don't have to worry about anything else.
    
In Swift, this is not possible due to static typing. We have to use the real thing or subclasses, or we extract a protocol and implement it in both the managed object and the test double.

With the experience I just made this week, I'd prefer protocols from the start. But sometimes this won't be possible because you use types incompatible with Objective-C. Then you're screwed.


## More Intricate Test Doubles in Swift 

Subclassing sounds like the first reasonable approach to provide test doubles. And it works just fine in some cases.

Subclasses can provide a custom initializer. But [as the documentation warns us][docs], we should not override quite a few methods, and we should stick to the designated initializer. The designated initializer depends on `NSEntityDescription` and `NSManagedObjectContext`; that's exactly the two things I would want to get rid of for creating test doubles. That's why I tried not to use the designated initializer and see how far I get.

Spoiler: It's perfectly possible to create instances without calling the designated initializer `init(entity:, insertIntoManagedObjectContext:)`. But you may surprised what actually happens under the hood and the problems you will run into.

    class TestThing: ManagedThing { 
      // override a few methods and provide canned responses

      var testComputationResult = 0
      override func computeSomethingSpecial() -> Int {
        return testComputationResult
      }
    }

    let one = TestThing()
    one.testComputationResult = 42
    let other = TestThing()
    other.testComputationResult = 99

Here's the surprising thing: now `one` and `other` will share the same memory address, so both will have a `testComputationResult`of 99 -- just as if you had two handles for the same object. This makes variations in behavior impossible because you can't set any switches through instance variables: changing one will affect all instances.

Only when you call `NSManagedObject`s designated initializer will you get separate objects.

Using the designated initializer with all the Core Data overhead didn't work well for me because the subclass's method didn't seem to be called. Using protocols as a means of abstraction would've been a good idea -- however, the protocol would have to be annotated with `@objc`, so it's impossible to reference `struct` types, for example. When I didn't annotate it with `@objc`,  I wasn't able to cast back and forth, so the tests didn't run at all.

Changing lots of my value types to classes just for the sake of making the protocol available to Objective-C and Core Data is a no-go. That violates the reasons I have cared about picking value types in the first place.

A weird way to get around the limitation of wrongly initialized managed object subclasses: create even more subclasses!

    class TestThing: ManagedThing { 
      // set some common variables as canned responses except
      // the one which varies between subclasses
    }

    class OneThing: TestThing {
      override func computeSomethingSpecial() -> Int {
        return 42
      }
    }

    class OtherThing: TestThing {
      override func computeSomethingSpecial() -> Int {
        return 99
      }
    }

    let one = OneThing()
    let other = OtherThing()

Now `one` and `other` will have different memory blocks. This is a very unexpected application of subclassing to encapsulate differences in behavior, aka the [Strategy][] pattern.

But it works, and it's only the tests which are affected. You should not make use of this during production at all because of the weird and unanticipated side effects these subclasses may cause.


[strategy]: http://www.oodesign.com/strategy-pattern.html
[docs]: https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/CoreDataFramework/Classes/NSManagedObject_Class/index.html
