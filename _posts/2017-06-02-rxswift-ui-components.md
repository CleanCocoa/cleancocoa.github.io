---
title: The 3 RxSwift Building Blocks of UI Components
created_at: 2017-06-02 15:25:00 +0200
tags: [ rxswift ]
comments: on
---

So here's what I learned so far about the building blocks of reactive UI components from peeking at the RxCocoa source.

## The 3 Building Blocks

UI components in general can have properties (read/write), input ports (read), and output ports (write). Classic UIKit/AppKit output ports would be delegate calls; classic input ports would be commands like `display(banana:)` that you probably write every day or so.

Translated to the world of RxSwift:

- `Observable` is the basic output sequence
- `Observer` is the basic consumer of input event sequences
- a combination of both fits mutable properties, the like `Variable` type, for example

Then there are special "traits" for UI bindings which guarantee to work on the main queue.
 
- `ControlEvent` is an `Observable` trait
- `UIBindingObserver` is an `Observer` trait
- `ControlProperty` has both traits's attributes and is an `Observable` sequence as well as an `Observer`

## Translating Known UIControl/NSControl Properties to RxSwift

For example, `NSTextField` and `UITextField` expose `.rx.text` which is a `ControlProperty<String>`. That means it's read-write. You can bind other sequences to this property and have the text field update its content; and you can observe user-generated changes as well.

The alpha value or translucency of a view component is a `UIBindingObserver`; similarly, `isEnabled` is a `UIBindingObserver`. [Look at `UIControl+Rx`](https://github.com/ReactiveX/RxSwift/blob/0b66f666ba6955a51cba1ad530311b030fa4db9c/RxCocoa/iOS/UIControl%2BRx.swift#L19) of RxCocoa for details. 

The reasoning I came up with: you usually want to change the alpha value or enabled state in reaction to some event, so it needs to be an `Observer` (or "sink") of sorts. You don't expect any view component to generate changes to these on its own terms. These things are toggles that a controller usually manipulates. 

So even though the underlying property of the UIKit/AppKit component is a `readwrite` (or `var`) property, it does not make too much sense to expose a `ControlProperty` in these cases. 

That doesn't mean you couldn't, thanks to KVO. Here's [a working but conceptually bad example](https://github.com/ReactiveX/RxSwift/pull/1270) of how to expose a view property in a reactive way that I wrote:

```swift
public extension Reactive where Base: NSScrollView {
    public var backgroundColor: ControlProperty<NSColor> {
        let source = self.observeWeakly(NSColor.self, "backgroundColor", options: [.initial, .new])
            // Skip nil values (which in practice does not happen, but KVO observe returns Optional)
            .filter { $0 != nil }.map { $0! }
            .takeUntil(deallocated)

        // `base` is a property of `Reactive` and, here, of type `NSScrollView`
        let observer = UIBindingObserver(UIElement: base) { (scrollView, newColor: NSColor) in
            scrollView.backgroundColor = newColor
        }

        return ControlProperty(values: source, valueSink: observer)
    }
}
```

As you see, the `ControlProperty` is composed of a `UIBindingObserver` and a regular `Observable` sequence. But there's a catch: `ControlProperty` requires your source sequence to adhere to a few conventional criteria. These criteria are not enforced through the types in this case. `ControlProperty` takes responsibility of adhering to certain characteristics but it's your job to make them happen. Let's talk about these rules in detail.

## The Rules of Using These 3 RxCocoa Types Correctly

### ControlEvent

The implementer (you!) has to provide an `Observable` that's safe and auto-completes on deallocation of the object. `ControlEvent` itself takes care of main queue scheduling if needed.

Quoting the Swift source of RxCocoa v3.5, `ControlEvent` ...

> - it never fails
> - it won't send any initial value on subscription
> - it will `Complete` sequence on control being deallocated
> - it never errors out
> - it delivers events on `MainScheduler.instance`

Most `Observable` sequences you build don't send initial values, but there are some that have replay behavior you should be aware of. 

For example, a `Variable("initial")` sends `.next("initial")` when subscription starts. A `BehaviorSubject<String>("inital")` does, too. But a `PublishSubject<String>()` doesn't send anything upon subscription. You can see that it doesn't require an initial value. `Observable.from(["initial"])` will start right away, too. 

## ControlProperty

The requirements of `ControlProperty` are similar to these of `ControlEvent`, but an initial value is expected, so the source has to `shareReplay(1)`.

Again, `ControlProperty` itself takes care of main queue scheduling.

> - it never fails
> - `shareReplay(1)` behavior
>     - it's stateful, upon subscription (calling subscribe) last element is immediately replayed if it was produced
> - it will `Complete` sequence on control being deallocated
> - it never errors out
> - it delivers events on `MainScheduler.instance`

## UIBindingObserver

The "soft" attributes (that is, not enforced through the typing system) of `UIBindingObserver` are simpler: it does not bind errors (errors will merely be logged). And if the incoming sequence is not on the main queue already, `UIBindingObserver` dispatches to the main queue asynchronously. 

From the call site, you don't have to do anything.

It's pretty easy to use:

```swift
public extension Reactive where Base: BananaView {
    public var size: UIBindingObserver<BananaView, Size> {
        // `base` is a property of `Reactive` and of type `BananaView`
        return UIBindingObserver(UIElement: base) { (bananaView, newSize) in
            bananaView.growVisibleBanana(to: newSize)
        }
    }
}
```

You pass in an object to `UIBindingObserver.init` that you want to change for incoming events. `UIBindingObserver` does the weak--strong-dance for you. It keeps a weak reference to the element only, so you don't end up with a retain cycle. When a `.next` event reaches the observer, and if the weakly referenced element still exists, a reference to the element is passed back into the closure where you can mutate the underlying property. 

If you create your objects using `Variable` type properties, you don't need much of this stuff, of course. But if you want to provide a reactive extension to a regular view component, this is how you can do.
