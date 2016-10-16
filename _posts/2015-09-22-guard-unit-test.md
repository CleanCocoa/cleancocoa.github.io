---
title: Using Guard in Unit Tests
created_at: 2015-09-22 12:17:14 +0200
kind: worklog
tags: [ swift, unit-test, guard ]
comments: on
---

I have just discovered how cool the new `guard` statement is to keep unit tests lean and shallow -- avoiding if-let nesting, that is.

Consider this test case:

    #!swift
    func testAssigningGroup_AddsInverseRelation() {
        
        insertFile() // Create new entities into the temporary context
        insertGroup()
        
        let file = soleFile()    // -> File?
        XCTAssert(hasValue(file))
        
        let group = soleGroup()  // -> Group?
        XCTAssert(hasValue(group))
        
        file?.group = group
        
        if let files = group?.files {
            XCTAssertEqual(files.count, 1)
            XCTAssert(files.anyObject() === file)
        }
    }

There are two optionals I have to deal with. That's why I throw in assertions to cover problems. I don't want to assert anything new here, though: the test helper `soleFile()` is valdiated in another test case already. But I need some kind of failure in case the Core Data test case goes nuts.

Consider this Swift 2 version of the test case:

    #!swift
    func testAssigningGroup_AddsInverseRelation() {
        
        insertFile()
        insertGroup()
        
        guard let file = soleFile() else {
            XCTFail("no file")
            return
        }
        
        guard let group = soleGroup() else {
            XCTFail("no group")
            return
        }
        
        // Precondition
        XCTAssertEqual(group.files.count, 0)
        
        file.group = group
        
        // Postcondition
        XCTAssertEqual(group.files.count, 1)
        XCTAssert(group.files.anyObject() === file)
    }

The guard-let pattern propagates the constant to the outer scope, as opposed to if-let. It's a great way to stay "shallow".

I also noticed that I missed to test preconditions. The shallow Swift 2 version revealed this: the test wasn't symmetric. There simply was nothing after the guard clause to verify or enforce an expected preconditional state.

Unwrapping optionals makes assertions easier. And failing to unwrap the optional will now produce a test failure among preconditions. I like that.
