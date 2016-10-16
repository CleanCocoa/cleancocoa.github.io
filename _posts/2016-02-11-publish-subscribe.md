---
title: "Publish and Subscribe — Decoupling Deep View Hierarchies from Event Handlers"
created_at: 2016-02-11 15:24:51 +0100
kind: worklog
tags: [ event, subscription, notification ]
comments: on
vgwort: http://vg01.met.vgwort.de/na/25bfecd273f24cf99f09630a8f5b6f91
---

Imagine a complex view with many sub components. This is more common in Mac apps where a window contains multiple panes, lists, graphs, whatever. How do you react to interactions 5 levels deep? Let's say you avoid massive view controllers which do everything on their own and want to encapsulate event handling outside the view hierarchy -- what should you do?

I experiment with a combination of event publishing and a presenter.

As a concrete example, here's the levels from the new Word Counter module I deal with:

- `AnalyticsWindowController`, displaying analytics for a set of apps of a certain date range; the container of it all
- `AnalyticsDetailsViewController` groups everything that's not the graph
- `ApplicationListController` shows a table view with app details
- `ApplicationCell` shows a single data point

The last in the hierarchy handles button presses and should dispatch an event. Keeping it generic, the naive and easy to test approach is to accept an optional closure (or a delegate for similar effects):

    #!swift
    var eventHandler: (() -> Void)?

That pattern works very well for direct setup -- but information doesn't bubble up the chain very well. I'd have to make each level a facade to the underlying levels, passing the eventHandler down from window to cell. That's quite an effort.

On top of that, when the controllers are created form Nibs, setting the value on the window controller can work, but when the window controller is a facade and delegates to its sub-view controllers before the Nib is loaded, boom, you end up with an exception. So I could accept an `eventHandler` on every level, but instead of passing it down with `didSet` property observers immediately (which can be too soon), I'd have to pass it along in `awakeFromNib`. That's even more entangled. 

## Publishing/Subscribing

I found a custom event publish/subscribe setup can help. Dispatch an event from the anywhere and the subscriber will be notified. Essentially, you abstract away the underlying object graph which otherwise has to be traversed to get from A to B. 

In accordance with the example, this is what a first implementation of a centralized publisher of this view module looks like:

    #!swift
    class AnalyticsWindowEventPublisher {
        
        var queue = dispatch_get_main_queue()
        var subscribers = [AnalyticsWindowEventSubscriber]()

        init() { }

        func publish(event: AnalyticsWindowEvent) {

            for subscriber in subscribers {
                dispatch_async(queue) {
                    subscriber.on(event)
                }
            }
        }

        func subscribe(newSubscriber: AnalyticsWindowEventSubscriber) {

            guard isNewSubscriber(newSubscriber) else {
                return
            }

            subscribers.add(newSubscriber)
        }

        private func isNewSubscriber(subscriber: AnalyticsWindowEventSubscriber) -> Bool {

            return subscribers.contains { $0 === subscriber }
        }

        func unsubscribe(subscriber: AnalyticsWindowEventSubscriber) {

            guard let index = subscribers.indexOf({ subscriber === $0 }) else {
                return
            }

            subscribers.removeAtIndex(index)
        }
    }

The subscriber protocol is simple, and you could do without one entirely and rely on closures instead if you want:

    #!swift
    protocol AnalyticsWindowEventSubscriber: class {
        func on(event: AnalyticsWindowEvent)
    }

Now there's room for some events:

    #!swift
    enum AnalyticsWindowEvent {
        case ChangeDateRange(NewDateRange)
        case UnselectApplication(ApplicationID)
        // ...
    }

I'd model the publisher as a singleton so you can better mock this in tests, but it also works when you provide a free function with a block underneath which you can replace during tests:

    #!swift
    func sharedEventPublisher() -> AnalyticsWindowEventPublisher {
        return sharedEventPublisherBlock()
    }
    
    let eventPublisher = AnalyticsWindowEventPublisher()
    var sharedEventPublisherBlock: () -> AnalyticsWindowEventPublisher {
        return eventPublisher
    }

What I use, in fact, is lazy properties. I simply inject test doubles as properties. That necessitates the `eventPublisher` isn't needed during initialization or else I'd use a singleton. 

    #!swift
    struct NewDateRange {
        let from: NSDate
        let until: NSDate
    }
    
    class DateRangePickerController: NSViewController, DisplaysDateRange {

        // ...
        
        lazy var publisher: AnalyticsWindowEventPublisher = eventPublisher

        @IBAction func applyDateRange(sender: AnyObject) {

            let newDateRange = NewDateRange(from: fromDatePicker.dateValue, 
                until: untilDatePicker.dateValue)

            publisher.publish(.ChangeDateRange(newDateRange))
        }
    }

This scales very well for the module I'm currently working on. I was afraid the subscribing wouldn't work in all cases, but as of yet it does. Publishing is easy and the publisher on `NSViewController`s can be replaced with a test double without any effort at all.

## Uni-directional flow with Presenters

Publishing events helps to decouple event handling from event dispatching. The actual event handling is simple.

I discovered that my presenter was too cluttered [last week][presenter], so I separated `EventHandler` from `Presenter`. The `EventHandler` takes commands and forwards them as instructions the `Presenter` understands.

Guess where the newly dispatched view events are handled. (Hint: it's not in the presenter.)

To make that work I don't let objects subscribe/unsubscribe themselves. Their parent component in the hierarchy takes care of that.

The `EventHandler` doesn't subscribe to events on its own. It is owned by a `ShowWindow` use case object that subscribes the `EventHandler` when the window opens and unsubscribes it when the window closes. This is necessary to insert some way to break the possible retain cycle of the publisher keeping a strong reference to the event handler which knows the presenter which knows the view.

    #!swift
    // The Subscriber
    class EventHandler {
        let presenter: Presenter

        init(presenter: Presenter) {
            self.presenter = presenter
        }
        
        func dateRangeDidChange(newDateRange: NewDateRange) {
            let dateRange = DateRange(newDateRange)
            presenter.changeDateRange(dateRange)
        }
    }

    extension EventHandler: AnalyticsWindowEventSubscriber {
        func on(event: AnalyticsWindowEvent) {
            guard case let .ChangeDateRange(dateRange) = event else { return }
            
            dateRangeDidChange(dateRange)
        }
    }
    
    // ... is subscribed elsewhere:
    class ShowAnalyticsWindow {
        // ...
        
        lazy var publisher: AnalyticsWindowEventPublisher = eventPublisher

        func showWindow() {
            analyticsWindowController.showWindow(self)
            eventHandler.showLast7Days()
            publisher.subscribe(eventHandler)   // ❗️
        }

        func windowWillClose() {
            publisher.unsubscribe(eventHandler) // ❗️
        }
    }

The fact that `EventHandler` in the above example doesn't have to worry about the events that make a subscription necessary is a huge relief. It only has to react to events. And eventually it'll get a proper name when the code base grows.

All of this was heavily inspired by my recent experiments with ReSwift, only I didn't want to introduce the framework into this project just for the sake of having a uni-directional way of presenting new information. Instead of the main store keeping the current state and broadcasting it to every interested party, the publisher forwards the latest event to its subscribers.

[presenter]: /posts/2016/02/private-function-collaborator/
