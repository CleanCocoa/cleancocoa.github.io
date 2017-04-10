---
title: "How to Unit Test Dispatching ReSwift Actions from RxSwift Observables"
created_at: 2017-02-07 11:01:57 +0100
kind: worklog
tags: [ rxswift, reswift, unit-test ]
vgwort: http://vg01.met.vgwort.de/na/aa6658aebaf548d6942f6633a718cddf
comments: on
---

Say you are like me and work with ReSwift for unidirectional data flow goodness and employ RxSwift for your reactive cravings. You want to dispatch a `ReSwift.Action` when an `RxSwift.Observable` signal produces a new value. How do you write tests for that wiring?

In code, it could look like this:

    #!swift
    someObservable
        .map { SomeAction($0) }
        .subscribe(onNext: store.dispatch($0))
        .addDisposableTo(disposeBag)

Testing that code requires a bit more understanding of both libraries than their respective documentation reveals.

Here's an example:

    #!swift
    import ReSwift
    
    struct AppState {
        var count = 0
    }

    typealias DefaultStore = Store<AppState>
    
    struct ChangingCount: ReSwift.Action {
        let value: Int
        init(value: Int) {
            self.value = value
        }
    }

Imagine a reducer that takes incoming `ChangingCount.value` and replaces `AppState.value` with that. Simple.

Now you want to dispatch a `ChangingCount` action when some signal produces a new value. You need to create an observer for the signal and store it somewhere. So a friendly helper class comes to mind:

    #!swift
    import RxSwift
    
    class Dispatcher {
        let store: DefaultStore
        let disposeBag = DisposeBag()
        
        init(store: DefaultStore) {
            self.store = store
        }
        
        func wireSignalToAction(signal: Observable<Int>) {
            signal.map(ChangingCount.init(value:))
                .subscribe(onNext: { [weak self] in
                    self?.store.dispatch($0) })
                .addDisposableTo(disposeBag)
        }
    }

And that's it. Now how can you make sure it does what it should do? What do unit tests look like?

Taking a look at `RxSwift`'s `TestObserver` and `TestScheduler`, you can record incoming events yourself and perform checks. To record events, you hijack a `ReSwift.Store`'s `dispatchFunction` with a closure that captures the incoming action. Using a `TestScheduler`, you can time production of events in the signal and keep track of the time the action comes in:

    #!swift
    import RxSwift
    import RxTest
    
    class DispatchingTests: XCTestCase {
        /// Replacement for your real reducer because you won't need it.
        struct NullReducer: ReSwift.Reducer {
            func handleAction(action: Action, state: AppState?) -> AppState {
                return AppState(value: -1)
            }
        }
        
        func testDispatchesValueChanges() {
            
            // Set up a signal. Subscription per default starts at time=200
            let scheduler = TestScheduler(initialClock: 0)
            let signal = scheduler.createHotObservable([
                next(300, 98),
                next(600, 12)
                ])
            
            // Set up the store and record matching actions.
            let storeDouble = Store(reducer: NullReducer(), state: nil)
            var actions = [Recorded<Event<ChangingCount>>]()
            storeDouble.dispatchFunction = {
                guard let action = $0 as? ChangingCount else { return () }
                // `return f()` where `f() -> Void` is just like `return ()`.
                // We need to return something to satisy the `-> Any`.
                return actions.append(Recorded(time: scheduler.clock, value: .next(action)))
            }

            // Configure the object under test and start emitting
            // values in virtual time.
            let dispatcher = Dispatcher(store: storeDouble)
            dispatcher.wireSignalToAction(signal)
            scheduler.start()

            XCTAssertEqual(actions, [
                next(300, ChangingCount(value: 98)),
                next(600, ChangingCount(value: 12))
                ])
        }
    }
