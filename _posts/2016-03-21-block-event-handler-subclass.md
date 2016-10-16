---
title: "Translate Optional Delegate Protocol Methods to Swift Block-based Event Handlers Using Nested Objects"
created_at: 2016-03-21 17:34:54 +0100
kind: worklog
tags: [ closure, locality ]
comments: on
---

Found this little gem in a library called [`NextGrowingTextView`](https://github.com/muukii/NextGrowingTextView), shortened to make the point clear with less code. It's a collection of [block-based event handlers](/posts/2016/01/comparison-block-based-api-delegate/) in a little class:

    #!swift
    public class Delegates {
        public var textViewDidBeginEditing: (NextGrowingTextView) -> Void
        public var textViewDidEndEditing: (NextGrowingTextView) -> Void
        public var textViewDidChange: (NextGrowingTextView) -> Void

        public var willChangeHeight: (CGFloat) -> Void
        public var didChangeHeight: (CGFloat) -> Void
    }

So there's a bunch of traditional delegate callbacks -- only they are not defined as protocol requirements but as properties of an otherwise state-less type.

You use it like this:

    #!swift
    let growingTextView: NextGrowingTextView

    growingTextView.delegates.textViewDidChange = { (growingTextView: NextGrowingTextView) in
        // Do something
    }

Many views I write have less than 5 event callbacks. But if you write a complex component with many interactions or lots of decision-making, then it might be a good idea to wrap these up in a `Delegates` sub-type:

    #!swift
    class BananaComponent: UIView {
        // ...
        let delegates = Delegates()
        
        class Delegates {
            // Favor "no-op" defaults over optionals
            var bananaComponentDidShow: (BananaComponent) -> Void = { _ in }
        }
    }
