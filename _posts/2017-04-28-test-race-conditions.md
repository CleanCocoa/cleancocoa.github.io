---
title: One Way to Solve Inexplicable Race Conditions in Tests
created_at: 2017-04-28 10:29:13 +0200
tags: [ testing, concurrency ]
comments: on
---

Today I wrote my first asynchronous test in [my latest project](http://zettelkasten.de/beta/) and suddenly, during the `waitForExpectations` run loop, a totally unrelated test's stuff crashed. To make things worse, it was part of an RxSwift subscription. No idea who kept that alive and where the `EXC_BAD_ACCESS` really originated.

It helped to make sure that no object references are kept around longer than necessary. In unit tests, this can mean to make your test doubles and collaborating objects optionals:

```swift
class BananaTests: XCTestCase {

    var banana: Banana!
    var tree: Tree!
    var monkey: Monkey!
    
    override func setUp() {
        super.setUp()
        tree = Tree()
        monkey = Monkey()
        banana = Banana(on: tree, eatenBy: monkey)
    }
    
    override func tearDown() {
        tree = nil
        monkey = nil
        banana = nil
        super.tearDown()
    }
}
```

If you just go the lazy route and make the objects properties of the `BananaTests` class, the objects will be kept around until all tests have finished and the `BananaTests` instance is being disposed of. With the setup--tear down dance here, we manage to get rid of references right after each test execution.

In case you wonder what the lazy version looks like where the latest references are kept alive:

```swift
class BananaTests: XCTestCase {

    var banana: Banana!
    let tree = Tree()      // <- uh oh
    let monkey = Monkey()  // <- guess who stays around ...
    
    override func setUp() {
        super.setUp()
        banana = Banana(on: tree, eatenBy: monkey)
    }
}
```

I'd love to tell you now that this solved my problem. (It didn't.) Instead, I want you to keep this in mind as a potential source of problems.  Tests for `NotificationCenter` subscribers will are a great example of problematic tests: the subscribers will continue to receive notifications when other test suites run.
