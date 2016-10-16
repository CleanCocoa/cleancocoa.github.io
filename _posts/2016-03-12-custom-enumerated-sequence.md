---
title: Creating a Cheap Protocol-Oriented Copy of SequenceType (with a Twist!)
created_at: 2016-03-12 11:13:35 +0100
kind: worklog
tags: [ protocol, swift, sequence ]
vgwort: http://vg01.met.vgwort.de/na/8a763d7b9ddc405d8361703dd9588698
comments: on
---

When you implement `CollectionType` or `SequenceType` in custom types, you get a lot for free with just a few steps, like being able to use `map` on the collection. 

Any `CollectionType` implementation comes with `enumerate()` which, when you iterate over the result, wraps each element of the collection in a tuple with its index:

    #!swift
    for (index, element) in ["a", "b"].enumerate() {
        print("\(index): \(element)")
    }
    // Prints
    //   0: a
    //   1: b

Implementing `CollectionType` requires you add a `startIndex` and `endIndex` of a type of your choosing.  I didn't use `Int` but my own `Index` type (`UInt > 0`, so its first value is 1, not 0) for these. I had expected `enumerate()` to pick that up. It didn't, because it's not tied to that type association.

Either I would `+1` the enumeration index and unwrap the resulting optional `Index`, or I could create my own `enumerate` that returns a tuple of `(Index, Element)` starting at `Index(1)` instead of `(Int, Element)` starting at 0.

The naive approach is simple:

    #!swift
    func enumerate() -> [(Index, Element)] {
        // Thanks to making the types explicit, the compiler knows
        // that the original `enumerate()` should be called.
        return self.enumerate().map { (index: Int, element: Element) in
            
            return (Index(index + 1)!, element)
        }
    }

But it's wasteful because you transform all elements at once to fill an array.  And it's not very ... _cool_. I wanted to try to reconstruct what the Swift developers did to better understand the language. So I did.

## Copying how `SequenceType` makes `enumerate()` possible

With a little fiddling I had this little breakthrough in constructing custom sequences. All this protocol-oriented programming stuff is rather mind-bending at times. I'm by no means a Swift wiz. When I look at the [Swift stdlib source code][swift] I'm always amazed how they came up with that stuff. So let's copy the Swift code in a very superficial way and see how to do the magic.

[swift]: https://github.com/apple/swift/blob/97d8f50af4978e54289377a0bd205e71a34529a2/stdlib/public/core/Sequence.swift

The key component is the protocol itself. 

    #!swift
    protocol IndexedSequenceType {

        typealias Sequence: SequenceType
        func enumerate() -> IndexedEnumeratedSequence<Sequence>
    }

The original `enumerate()` doesn't return an array but an `EnumetareSequence`. So we need an `IndexedEnumeratedSequence`, right? This is how I started. But it isn't much of a solution, yet: instead of having just a protocol to use right away, I have a protocol _and_ need to write an `IndexedEnumeratedSequence`.

Copying heavily from [`EnumerateSequence` from the Swift 2.x stdlib](https://github.com/apple/swift/blob/f4310256386b9a578310177752a26cf0b0684f3a/stdlib/public/core/Algorithm.swift#L196), the result is this:

    #!swift
    struct IndexedEnumeratedSequence<Base : SequenceType>: SequenceType {

        let _base: Base

        init(_ base: Base) {
            self._base = base
        }

        func generate() -> IndexedEnumeratedIterator<Base.Generator> {
            return IndexedEnumeratedIterator(_base.generate())
        }
    }

So now I have a type-erased struct that gets rid of the associated type of the `SequenceType` protocol through generics. _And_ I need to write an `IndexedEnumeratedIterator` next. This was the fun part which I'd have started with did I know where all of this would lead:

    #!swift
    struct IndexedEnumeratedIterator<Base: GeneratorType>: GeneratorType {

        var _base: Base
        var _count: Index

        internal init(_ base: Base) {
            self._base = base
            self._count = Index.first // equals `Index(1)!`
        }

        typealias Element = (offset: Index, element: Base.Element)

        mutating func next() -> Element? {
            guard let b = _base.next() else { return nil }
            defer { _count = _count.successor() }
            return (offset: _count, element: b)
        }
    }

If you follow along with where Swift 3 is heading, you'll notice I call the type "Iterator" instead of "Generator" which Swift 2.x was using. That's by design so I don't have to rename my types in a few months. The [`EnumerateGenerator` of Swift](https://github.com/apple/swift/blob/f4310256386b9a578310177752a26cf0b0684f3a/stdlib/public/core/Algorithm.swift#L161) looks very similar.

With the `IndexedEnumeratedSequence` and `IndexedEnumeratedIterator` as wrappers to make the type requirements concrete in place, I now can use `IndexedSequenceType`.

A type-erased generic `IndexedSequence` works like this:

    #!swift
    struct IndexedSequence<Base: SequenceType>: IndexedSequenceType {

        let _base: Base

        init(_ base: Base) {
            self._base = base
        }

        func enumerate() -> IndexedEnumerateSequence<Base> {

            return IndexedEnumerateSequence(_base)
        }
    }

Okay, so we're set. The compiler doesn't complain here. Will it complain for the real thing?


## Being happy that it works

The actual code is related to my [table columns lenses][lens]. Adding the implementation is simple since the lenses are of type `CollectionType` already:

    #!swift
    extension ColumnLens: IndexedSequenceType {

        func enumerate() -> IndexedEnumerateSequence<ColumnLens> {

            return IndexedEnumerateSequence(self)
        }
    }

Thanks to these lines, I can now enumerate columns and their indexes to quickly create `NSTableColumn`s from them with proper index-based identifiers:

    #!swift
    let tableColumns: [NSTableColmn] = table.columns.enumerate()
        .map { (index: Index, column: Column) in
            
            let tableColumn = NSTableColumn(identifier: "\(index)")
            tableColumn.title = column.heading.content

            return tableColumn
        }

The `IndexedSequenceType` protocol isn't very powerful and doesn't even inherit from `SequenceType` right now (because I don't need that, although this would be desirable). Still I'm very happy about the result and that I was able to reconstruct the sequence and enumeration code from Swift's stdlib. 

With a few more of these exercises, maybe I'll be able to think in terms of use-cases that call for PATs (protocols with associated types) and type-erasing during the modelling phase.

Thinking about the benefit over the naive approach though, I then wondered if there'd be another way. Maybe ...

## Then being annoyed that lazy collections work just as well

The initial problem with the naive mapping was that the whole sequence would be enumerated once to transform the values and return an array of the result. That's quite a waste when you intend to use a modification of an algorithm.

A similar result of all the above can be achieved by using `lazy`, only without these extra types:

    #!swift
    extension ColumnLens {
        
        func enumerate() -> LazyMapSequence<EnumerateSequence<ColumnLens>, (Index, Column)> {

            // Explicit type annotations to aid the compiler
            let enumerator: EnumerateSequence<ColumnLens> = self.enumerate()
            let result = enumerator.lazy.map { (index: Int, element: _Element) in
                return (Index(index + 1)!, element)
            }

            return result
        }
    }

This is like returning an enumerable that carries the modification of the algorithm with is. Instead of calling `map` _N_ times in advance, you get a very low-footprint object that stored `map`'s closure and calls it only when requested during follow-up sequence operations. The results are not cached, though, so iterating the sequence twice will cost twice as much.

Sure, the return type is hideous, but a `typealias` can do wonders.

[lens]: /posts/2016/02/lens-data-interface/

## But which is better?

Performance-wise, I don't know for sure, but `lazy` does a slightly better job in the following tests where 100000 elements are enumerated -- and, for the sake of accessing the result, reduced into a single string:

    #!swift
    struct Nums: CollectionType {

        subscript(index: Int) -> Int {
            return index
        }

        let startIndex: Int = 0
        let endIndex: Int = 100000

        func enumerateLazy() -> LazyMapSequence<EnumerateSequence<Nums>, (TableFlip.Index, Int)> {

            let enumerator: EnumerateSequence<Nums> = self.enumerate()
            let result = enumerator.lazy.map { (index: Int, element: _Element) in
                return (TableFlip.Index(index + 1)!, element)
            }

            return result
        }

        func enumerateIndexed() -> IndexedEnumeratedSequence<Nums> {

            return IndexedEnumeratedSequence(self)
        }
    }


    class IndexedSequenceTypeTests: XCTestCase {

        func testPerformance_Type() {

            let nums = Nums()

            self.measureBlock {
                _ = nums.enumerateIndexed().map { index, element in
                    "\(index): \(element)"
                    }.reduce("", combine: { memo, element in
                        "\(memo)\(element)"
                    })
            }
        }

        func testPerformance_Lazy() {

            let nums = Nums()

            self.measureBlock {
                _ = nums.enumerateLazy().map { index, element in
                    "\(index): \(element)"
                    }.reduce("", combine: { memo, element in
                        "\(memo)\(element)"
                    })
            }
        }
    }

The typed solution is about 0.01s slower with these settings. When you increase the size of the collection to 1000000, it takes about 4.0s where the lazy variant takes 3.9s. So going with `lazy` sounds like a good idea if the type doesn't provide any additional value.

I put the `lazy` enumeration into a protocol and called the method `indexedEnumerate()` for clarity:

    #!swift
    protocol IndexedSequence: SequenceType {

        func indexedEnumerate() -> LazyMapSequence<EnumerateSequence<Self>, (Index, Self.Generator.Element)>
    }

    extension IndexedSequence {

        typealias IndexedEnumeratedElements = LazyMapSequence<
                EnumerateSequence<Self>, 
                (Index, Self.Generator.Element)>

        func indexedEnumerate() -> IndexedEnumeratedElements  {

            let result = enumerate().lazy.map { 
                (index: Int, element: Self.Generator.Element) in

                // Enumerator index always starts at 0.
                return (Index(index + 1)!, element)
            }
        
            return result
        }
    }
    

Well, should the time come where I need the previous stuff around `IndexedSequenceType` again, I can come here and copy the code back into the project later.
