---
title: Adding RxSwift Ports to Resolve UI Inconsistencies
created_at: 2017-05-16 21:09:00 +0200
keywords: [rxswift, ui]
---

I am currently struggling to write my user interface layer (Mac) in a way that works well with RxSwift (the reactive library). Meanwhile, the whole app's state is maintained by ReSwift (the unidirectional data flow library). Both work well together, but I made some weird choices initially.

For example, I'd write Presenters to subscribe to store changes, assemble a static View Model, then pass that on to the View. That's all nice and imperatively object-oriented.

But then I wrote the actual view controller (which implements the View protocol) to simply update the UI components like I always did. That worked well, it seemed, up until it suddenly began to cause trouble. At the same time, I'd use RxSwift to publish user-initiated changes as events.

Input and output qdhered to different paradigms. In the case of a `NSTextField`, you can rely on the `.rx.text` property to produce a change signal when the user types. But when you set the `stringValue` manually, it won't. That was a very nice "feature," by accident!, because now programmatically changing the text field didn't fire a change event again. Hooray!

Weeks later, I run into problems with edge cases where sometimes the text field should not be updated for some reason, and sometimes where events should not be fired. The mixture of imperative and reactive input VS output got in the way. The solution, in this case, was to use a naive implementation of a reactive view model. I'd keep both the imperative `display(string:)` input port and the reactive `searchTermChange` output port around. But instead of relying on the text field as mediator, I'd add the view model as the _local source of truth._

- When the presenter wants to `display(...)` something, the view controller receives the value and either changes a `RxSwift.Variable` or call `.on(next(...))` for a `RxSwift.PublishSubject`. Both types are great to pipe changes through, each with a different set of strengths.
- Either way the value change will be observed and trigger an update to the text field's contents in all cases that apply.
- Typing into the text field triggers a `searchTermChange`, as already mentioned.
- Text field changes that are identical to the view model, though, will be ignored! This way the binding of view model to text field will not emit a change event.

Ignoring identical values from two sources typically looks like this (in my view controllers):

```swift
// Typing is the source stream of itnerest
textField.rx.text
    
    // Combine typed and programmatically changed 
    // values into a single stream
    .withLatestFrom(viewModel) { ($0, $1) }
    
    // Select typed-in values if they are different
    .filter { new, old in
        new != old }
    .map { new, _ in new }
    .distinctUntilChanged()
    
    // Emit change event
    .bindTo(searchTermChange)
    .addDisposableTo(disposeBag)
```

It's awkward that I had to add another RxSwift-based property at first, but in the end it made sense to unify input and output so I can combine both kinds of streams and work with them more easily.

Figuring out when an observable emits events and why or how combinations of streams produce results still is very hard. I think I am developing a fist kind of intuition, though, and when the refactoring has proven to be useful, I'll distill the lessons into blog posts, of course.