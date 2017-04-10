---
title: "Non-Obvious Swift: Defer"
created_at: 2017-03-26 09:35:07 +0200
kind: worklog
tags: [ swift ]
comments: on
---


The following code works as expected:

    #!swift
    class FooCollection {
        private var items = [Foo]()
    
        func removeAllItems() -> [Foo] {

            defer { items.removeAll() }
            return items
        }
    }
    
But do you know what "expected" means in this case?

As a reader, you assume the author had an intention. You look for the _mens auctoris_ and are an overall benevolent reader, I hope. Presupposing said intention, you may assume that it does something special if you put the call to `items.removeAll()` in a `defer` block.

The `removeAllItems` method returns an array of items. If the internal collection was empty when the `return` statement is reached, that'd be pointless, wouldn't it? Since your benevolent, you assume that the author isn't stupid and that it does indeed return a non-empty collection in some cases.

Say the author had added a documentation line:

    #!swift
    /// - returns: Array of items that were removed.
    func removeAllItems() -> [Foo] { // ... }

Now that should tip the scale! So the internal collection of items is returned _and afterwards_ emptied. Aha! How clever!

{% include figure.md src="/assets/blog/meme_like-a-sir.png" caption="Ze true connoisseur of Swift appreciates ze brevity of <code>defer</code>" %}

As a critical reader, you should be able to solve the puzzle _and_ call the author names. Because, why make it so non-obvious to the reader? Why the guessing that either requires manual (or unit-) testing or the Swift doc (and trust in its truth) to verify the assumptions?

I was curious about the outcome of this approach so I just tried it before writing this up. Then I deleted the \"\"\"clever\"\"\" code and replaced it with what I had before:

    #!swift
    class FooCollection {
        private var items = [Foo]()
    
        func removeAllItems() -> [Foo] {
            let removedItems = items
            items.removeAll()
            return removedItems
        }
    }

I prefer this any day. I hope you do, too.
