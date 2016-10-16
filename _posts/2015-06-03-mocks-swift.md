---
title: State of Mocking in Swift Tests
created_at: 2015-06-03 16:09:27 +0200
kind: worklog
tags: [ swift, testing, stub, mock ]
shared_image: "blog/201506031611_strawman-t.jpg"
vgwort: http://vg08.met.vgwort.de/na/b0c1b31ef4f246c093a68037a32a9874
comments: on
---

Mocking and stubbing in tests is useful to verify behavior and replace collaborating objects with simpler alternatives. Since Swift is statically typed, you can't use a mocking library anymore. But it's trivially simple to write [simple mock objects as subclasses in place](http://nshipster.com/xctestcase/):

    #!swift
    class Bar { func bar() {} }
    
    func foo(bar: Bar) {
        bar.bar()
    }
    class FooTest: XCTestCase {
        func testFoo() {
            class MockBar: Bar { /* ... */ }
            let barDouble = MockBar()
        
            let baz = foo(barDouble)
        
            XCTAssertNotNil(baz)
        }
    }
    
You can declare subclasses in the scope of the test method and override only the minimum necessary parts.

I like to take the additional safety measure to exclusively descend from [Null-Objects][nop] in such cases. If `foo()` called another method on its `Bar` parameter than expected, we end up with the real behavior and real side-effects. The Null-Object pattern ensures the parent class does nothing for every method, not just the one we override in a specific test.

{% include figure.md src="/assets/blog/201506031611_strawman.jpg" alt="strawman" caption="Photo Credit: <a href=\"https://www.flickr.com/photos/earl258/2448032465/\">稻田 ricefilm Strawman</a> by <a href=\"https://www.flickr.com/photos/earl258/\">earl258</a>. License: <a href=\"https://creativecommons.org/licenses/by-nc/2.0/\">CC-BY-NC 2.0</a>" %}

Using subclasses private to a method is clever, but it doesn't cut it. You can write stubs easily, but you cannot write [mock objects](http://martinfowler.com/articles/mocksArentStubs.html#TestsWithMockObjects) which record incoming calls for verification without a lot of hassle.

`XCTestExpectation` would work to verify calls -- but it's meant for asynchronous tests. Calling `waitForExpectationsWithTimeout(0)` just to verify if a synchronous expectation is fulfilled is overkill. (Using larger timeouts is a bad idea as it slows down your test suite without any benefit.)

Also, I get segmentation faults all the time:

    #!swift
    func testFoo() {
        let expectation = expectationWithDescription("called")
        class MockBar: Bar {
            private override func bar() {
                expectation.fulfill() // comment-out this line to not raise segfaults
            }
        }
    
        let barDouble = MockBar()
    
        foo(barDouble)
    
        waitForExpectationsWithTimeout(0) { error in }
    }

Same for booleans, which otherwise sound like a reasonable alternative:

    #!swift
    func testFoo() {
        var expectation = false
        
        class MockBar: Bar {
            private override func bar() {
                expectation = true
            }
        }
    
        let barDouble = MockBar()
    
        foo(barDouble)
    
        XCTAssertTrue(expectation)
    }

So we end up with the more clumsy variation of boolean properties:

    #!swift
    func testFoo() {
        class MockBar: Bar {
            var didCallBar = false
            private override func bar() {
                didCallBar = true
            }
        }
        
        let barDouble = MockBar()
        
        foo(barDouble)
        
        XCTAssertTrue(barDouble.didCallBar)
    }

This sucks, but it works.

Meanwhile, I [filed a Radar](https://openradar.appspot.com/21220076) and will now figure out if this is worth changing `struct`s to `class`es in production code. Because `struct`s cannot be sub_classed_.

[nop]: http://en.wikipedia.org/wiki/Null_Object_pattern
