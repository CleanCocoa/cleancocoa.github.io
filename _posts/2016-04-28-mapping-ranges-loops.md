---
title: Replacing Loops with Mapped Ranges 
created_at: 2016-04-28 11:31:16 +0200
kind: worklog
tags: [ swift, functional-programming ]
comments: on
---

During the last weeks, I tried to use `map()` even when the conversion wasn't straightforward to learn how it affects reading code. I found that some applications of `map()` were dumb while I liked how others turned out to read. The functional programming paradigm is about writing code in a declarative way, which has its benefits.

Take this, for example:

    #!swift
    extension Int {    
        func times<T>(f: () -> T) -> [T] {
            var result = [T]()
            
            for _ in 0..<self {
                result.append(f())
            }
            
            return result
        }
    }
    
    10.times { "foo" }
    // => ["foo", "foo", "foo", "foo", "foo", "foo", "foo", "foo", "foo", "foo"]

Looping over the range works well and is the way of the future to use for-loops from A to B, but a few months ago I thought this was clever already.

The method has 3 parts:

1. Declare a mutable array
2. Use (mutate) the array
3. Return the array

When I see mutable arrays used like this, I see `map`, though:

    #!swift
    extension Int {    
        func times<T>(f: () -> T) -> [T] {
            return (0..<self).map { _ in f() }
        }
    }

The result is a bit more terse. But unlike the mutable array from above, you do not wonder what will happen to the array, why it is mutable (`var` instead of  `let`) etc. Instead, you see that there's a range and that each element in the sequence is transformed ("mapped") to something else. 

The `return (0..<self).map` part already tells most of the story: you can expect an array with as many elements as `self` indicates. To guess what the contents will look like, you'll have to take a look at the block.

Compared this to the example from above. There, you knew an array is involved. Then there was a loop. Then the array was changed with every iteration. You had to read 3/4 of the method body to fathom what the result will be.

I am afraid to go too far down the rabbit hole of functional thinking, thus rendering my code harder to comprehend. This example is not _that_ bad I think. But when I prefer mapping the _N_ elements in a range to produce an array of size _N_, I think I already lost some people in my team. What's idiomatic Swift and thus to be expected from co-workers and what's too much?

