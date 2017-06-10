---
title: Adding last(where:) in Swift
created_at: 2017-06-10 09:54:34 +0200
tags: [ swift ]
comments: on
---

For quite a while I didn't notice `Sequence.first(where:)` exists. It's like `first`, only with a condition. I have now happily migrated from my self-baked `findFirst` to this method -- only to find out today that there's not `last(where:)` equivalent.

Makes sense at first, since `Sequence` is not stride-able backwards. But `BidirectionalCollection` is, and thus Array.

In a huge collection of stuff, it's probably too costly to simple call `array.reversed().first(where: myPredicate)`. Here, not even `lazy` would help since `reversed()` returns an Array in the new order instead of a `ReversedBidirectionalCollection<LazyCollection<Whatever>>` or similar. So I'm cautious and prefer to enumerate backwards.

`BidirectionalCollection` has an `indices` property. In case of arrays, that's a `CountableRange<Int>` which can be reversed far more cheaply. And the `Indices` associated type of `BidirectionalCollection` supports `reversed()`, too, so we can generalize to this:

```swift
extension BidirectionalCollection
where Self.Indices.Iterator.Element == Self.Index {
 
    func last(where predicate: (Self.Iterator.Element) -> Bool) -> Self.Iterator.Element? {

        for index in self.indices.reversed() {
            let element = self[index]
            if predicate(element) {
                return element
            }
        }

        return nil
    }
}
```

This iterates backwards without reversing the whole collection at first. Works with arrays and other interesting types like [Ole Begemann's `SortedArray`](https://github.com/ole/SortedArray).

By the way: why on earth would `BidirectionalCollection.Indices.Iterator.Element` be allowed to ever not equal `BidirectionalCollection.Index`? Without the "where" clause, the compiler will complain about the subscript: "Cannot subscript a value of type 'Self' with an index of type 'Self.Indices.Iterator.Element'".
