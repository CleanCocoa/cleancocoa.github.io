---
title: "Testing Shorthand: Use a Subclass of the Real Object Under Test"
created_at: 2015-09-24 16:09:03 +0200
kind: worklog
tags: [ testing, swift ]
shared_image: "blog/201506031611_strawman-t.jpg"
vgwort: ident
comments: on
---

I just found a shortcut to use [Dependency Injection][di] less during tests.

In ["Unit Testing in Swift 2.0"](https://realm.io/news/jorge-ortiz-unit-testing-swift-2/), Jorge Ortiz adapts solid testing principles to Swift. If you don't have a strong testing background, make sure to watch his talk for key insights into the _How_ using Swift!

He uses mocks for the object under test. Which is clever, because you can hook into its behavior.

In principle, Jorge does this:

    #!swift
    // Product target
    class BananaClient {
    
        func fetchBananas() {

            let serverData: NSData = ...
            // ... do something here ...
            
            do {
                if let bananasData = try parseJSONData(serverData, options: []) as? [NSDictionary] {
                    // complete fetching
                }
            } catch {
                // handle error
            }
        }
        
        func parseJSONData(data: NSData, options opt: NSJSONReadingOptions) throws -> AnyObject {
            return try NSJSONSerialization.JSONObjectWithData(data, options: opt)
        }
    }
    
    // Testing target
    
    class MockBananaClient: BananaClient {
        var parseJSONDataInvoked = false
    
        override func parseJSONData(data: NSData, options opt: NSJSONReadingOptions) throws -> AnyObject {
            parseJSONDataInvoked = true
            return []
        }
    }
    
    class BananaClientTests: XCTestCase {
        
        let client = MockBananaClient()
        
        // ...
    }

He does not create a [mock object][mock] of a dependency of `BananaClient`, injecting it into a `BananaClient` like we used to. He mocks the `BananaClient` itself to verify JSON parsing was or wasn't invoked.

_Usually_ I'd go and create a very small `JSONParser` which wraps the calls to the Cocoa API. I'd then provide a replacement for this parser in my test suites and verify its method was or wasn't called:

        #!swift
        // Product target
        class BananaClient {
    
            lazy var parser = JSONParser()
            
            func fetchBananas() {

                let serverData: NSData = ...
                // ... do something here ...
            
                do {
                    if let bananasData = try parseJSONData(serverData, options: []) as? [NSDictionary] {
                        // complete fetching
                    }
                } catch {
                    // handle error
                }
            }
        
            private func parseJSONData(data: NSData, options opt: NSJSONReadingOptions) throws -> AnyObject {
                
                return try parser.parse(data, options: opt)
            }
        }
        
        class JSONParser {
            
            func parseJSONData(data: NSData, options opt: NSJSONReadingOptions) throws -> AnyObject {
                
                return try NSJSONSerialization.JSONObjectWithData(data, options: opt)
            }
        }
    
        // Testing target
    
        class MockParser: JSONParser {
            var parseInvoked = false
    
            override func parse(data: NSData, options opt: NSJSONReadingOptions) throws -> AnyObject {
                parseInvoked = true
                return []
            }
        }
    
        class BananaClientTests: XCTestCase {
        
            let client = BananaClient()
            let parserDouble = MockParser()
            
            func setUp() {
                
                super.setUp()
                
                client.parser = parserDouble
            }
            
            // ...
        }

That still works really well for dependencies which encapsulate complex sequences you don't want to have in another object.

But wrapping a one-liner in a class just to isolate that single method call?

Jorge's little trick here is good to have in your testing toolkit.

[di]: https://en.wikipedia.org/wiki/Dependency_injection
[mock]: http://martinfowler.com/articles/mocksArentStubs.html
