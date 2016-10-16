---
title: "Refactoring Legacy Code: Replace Free Function in Tests, Even Swift's Own Functions"
created_at: 2015-07-29 10:38:47 +0200
kind: worklog
tags: [ legacy-code, testing, refactoring, swift ]
image: 201507291046_wallhole.jpg
comments: on
---


Refactoring Legacy Code is hard. There are a few safe refactorings you can do with caution. But most chirurgical cuts require you to put the code in a test harness first to guard against regression.

With C and Swift, you can create free functions as part of your app. To verify your objects use that function, you need to find a way to insert a test double.

It's possible to inject a different implementation for C functions during tests. Swift poses a different challenge, though.

When you have a free function in Swift and want to replace the default behavior with a mock object or a stub, here's the steps you need to take:

1. Refactor the function to delegate to a global closure in a variable.
2. In your test suite, copy the default closure during `setUp()`.
3. Replace the closure with a test double.
4. Ensure to set the closure back to its default during `tearDown()`.

This could look like the following:

    #!swift
    var fetchDataFromServerBlock: (ServerConnection) -> TheData = { conn in
        // ...
        return TheData(rawData: conn.data())
    }

    func fetchDataFromServer(conn: ServerConnection) -> TheData {
        return fetchDataFromServerBlock(conn)
    }

And in tests:

    #!swift
    class DataManglerTests: XCTestSuite {
        var originalFetchBlock: ((ServerConnection) -> TheData)!

        func setUp() {
            super.setUp()
            originalFetchBlock = fetchDataFromServerBlock
        }
        
        func tearDown() {
            fetchDataFromServerBlock = originalFetchBlock
            super.tearDown()
        }
        
        let aTheDataStub = TheDataStub()
        
        func testManglingFetchesData() {
            let mangler = DataMangler()
            var didFetch = false
            fetchDataFromServerBlock = { _ in 
                didFetch = true
                return aTheDataStub
            }
            
            mangler.mangle()
            
            XCTAssert(didFetch)
        }
        
        class TheDataStub: TheData {
            // ...
        }
    }

To make this even more robust, consider adding a `defaultFetchDataFromServerBlock` to switch implementations during tests:

    #!swift
    var fetchDataFromServerBlock = defaultFetchDataFromServerBlock
    let defaultFetchDataFromServerBlock: (ServerConnection) -> TheData { conn in
        // ...
        return TheData(rawData: conn.data())
    }

And in tests:

    #!swift
    class DataManglerTests: XCTestSuite {
        
        func tearDown() {
            fetchDataFromServerBlock = defaultFetchDataFromServerBlock
            super.tearDown()
        }
        
        // ...
    }

Swift is merely a year old. There won't be much legacy code written in Swift to deal with. But this technique applies to free functions in Objective-C as well: delegate to blocks and replace blocks during tests.

You can use this technique to test free functions from Swift's standard library, too: override the method in your module, then delegate to the original with the `Swift` module prefix. If you want to see an example, Nikolaj Schumacher wrote [a Gist to replace `precondition`](https://gist.github.com/nschum/208b3dde43785afd439a) during tests.

Check out Michael Feather's "[Working Effectively with Legacy Code](http://amzn.to/1DO97W7)" to learn more about safe refactorings and putting legacy code under test.

---

Picture: "[Inca](https://www.flickr.com/photos/7361805@N05/6590234331)" by Ajar on flickr. License: [CC-BY-SA](https://creativecommons.org/licenses/by-sa/2.0/).
