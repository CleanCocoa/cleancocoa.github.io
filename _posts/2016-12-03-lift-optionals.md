---
title: "Lifting Into a New Type: My first \"Idiomatic\" RxSwift Unit Test"
created_at: 2016-12-03 08:57:44 +0100
kind: worklog
tags: [ rxswift, testing ]
comments: on
---


I dabble with RxSwift right now. Figuring out how to write tests for pure functions and reactive code, I tried to write an assertion for incoming events:

    #!swift
    let expected: [Recorded<Event<URL?>>] = [next(0, nil), next(0, url)]
    XCTAssertEqual(observer.events, expected) 

Doesn't compile because equating arrays with `Optional<URL>` in them won't. In other words, `[Recorded<Event<URL>>]` (non-optional) would work.

I tries to figure out how to make this equatable, but then I stopped -- why not lift the `URL?` into its own type? After all, the signal I'm observing is a path settings change. `nil` is allowed to signify removing the setting, like when you reset to defaults.

Lifting this into its own "either-or" type, also known as encapsulating the information in an enum, I end up with this:

    #!swift
    enum URLChange {
        case set(URL)
        case unset
    }

Making this equatable is simple. And the tests work, too. For the record, here's the idiomatic Rx test:

    #!swift
    func testArchiveURLChanges_RxVersion() {

        let adapter = SettingsAdapter(defaults: UserDefaults.standard)
        let url = URL(fileURLWithPath: "/a/url/", isDirectory: true)
        
        let disposeBag = DisposeBag()
        let scheduler = TestScheduler(initialClock: 0)
        let observer = scheduler.createObserver(URLChange.self)

        adapter.archiveURLChange()
            .subscribe(observer)
            .addDisposableTo(disposeBag)

        defaults.set(url, forKey: "archiveUrl")

        let expected: [Recorded<Event<URLChange>>] = [
            next(0, .unset),   // Initial value for fresh defaults
            next(0, .set(url))
        ]
        XCTAssertEqual(observer.events, expected)
    }

And the traditional, asynchronous version I used before:

    #!swift
    func testArchiveURLChanges() {

        let adapter = SettingsAdapter(defaults: UserDefaults.standard)
        let ex = expectation(description: "Reports back result")
        let url = URL(fileURLWithPath: "/a/url/", isDirectory: true)
        let disposeBag = DisposeBag()
        
        adapter.archiveURLChange()
            .subscribe(onNext: { if $0.url == url { ex.fulfill() } })
            .addDisposableTo(disposeBag)

        defaults.set(url, forKey: "archiveUrl")

        waitForExpectations(timeout: 0.2, handler: nil)
    }

I'm not too happy with the verbose setup of the Rx test case. But I'm refactoring the setup into -- wait for it -- the `XCTestCase.setUp()` method.

For simple assertions, the async variant would work just as well. Recording longer sequences of events with some timing is interesting, though I didn't need that, yet, to test-drive my operators. 
