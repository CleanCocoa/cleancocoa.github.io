---
title: The Swift Developer's Cookbook Review
created_at: 2016-03-29 09:10:47 +0200
kind: worklog
tags: [ review, swift ]
image: 201603290859_cookbook.jpg
vgwort: http://vg01.met.vgwort.de/na/f76780d6d0a84d0d9cbe3d207b92cadb
comments: on
---

{% include figure.md src="/assets/blog/2016/201603290859_cookbook.jpg" alt="photo of the ebook" url="http://www.amazon.com/Developers-Cookbook-Content-Program-Library/dp/0134395263/ref=as_li_ss_tl?ie=UTF8&qid=1459235590&sr=8-1&keywords=swift+developers+cookbook&linkCode=ll1&tag=chritietwork-20&linkId=35655d685eb281d01543bf74c48ffad9" caption="Keeping the e-book handy on a tablet stand works great; copy and paste out of the PDF not quite so much" %}

Although I knew there's a "cookbook" book series, I haven't read any of these before I read [Erica Sadun's *The Swift Developer's Handbook*][cb]. The code listings are called "recipes", and most of the time rightly so: I bet I copied about a dozen of the recipes into my own handy code snippet index to extend Swift's capabilities with super useful things like [safe Array indexes][arr].

I am under the impression that I am quite an expert at Swift after about 2 years of daily practice. I know what I still don't know. In Erica's book, only little of this stuff was addressed. It's not a beginner's guide, either, though. It is aimed at programmers that want to grasp the language. There're no exercises for the reader. This book is a handbook. You use it to get a deep understanding of the language.

The table of contents reflects this fact:

1. **Welcome to Modern Swift**: migrating Swift code, REPL, scripting, command line
2. **Printing and Mirroring**: how mirrors and debug output relate to printing, Quick Look support, and header doc syntax (I'm always amazed how much low-level printing features are available which I don't need in my work)
3. **Optionals?!**: unwrapping Optionals first and foremost
4. **Closures and Functions**: anatomy of a function, tuples, currying
5. **Generics and Protocols**: how these two overlap and require each other
6. **Errors**: assertions and throwing, 7 Swift error rules, throwing errors with context
7. **Types**: reference vs value types, enums, option sets, classes, property observers
8. **Miscellany**: custom operators, String and sequence utilities

Although the topics become more complex in later chapters, I didn't find the order to be very important. The chapter on errors could've come later or earlier in the book, for example. A recipe from the "printing" chapter used a custom operator. -- Again, this is a handbook, not a learning guide.

The term "cookbook" fits nicely. Not only are code listings called "recipes", but the kind of listing really are copy-and-paste usable for you and your work. The explanations don't go deep; there's no discussion about how the compiler works. But they go deep enough so you have a criterion to decide: should you use a `final class` for optimized access, or should you use a `struct` value type instead?

I got a review copy of this book for free, which I'm thankful for because of its nature to be a handbook to look up something. Its recipes surprised me since I don't know a lot about using sequence types in clever ways, for example. I doubt I would've bought it, though, because the topics are so general. Still I love that I've learned some new details.

This makes giving a recommendation very hard. 

* If you need a guide to learn programming for Mac or iOS, this is the wrong book. Erica doesn't touch Cocoa (AppKit or UIKit) programming. This book is fully centered around the language itself. Except maybe 3 recipes that cast `String` to `NSString` and thus require the Foundation library, everything you can learn from this book applies to Swift developers on any platform -- be it coders using a Mac, Windows, or Linux machine.
* If you want deep expert-level insights into how you can use associated types in protocols and lift them to generics so you can build custom types that rely on these -- i.e. how `SequenceType/AnySequence` relate to one another --, this book isn't for you, either. You won't become a wizard of functional programming, either. Or application modeling.

It's aimed at people looking to get past literal translation from Objective-C patterns to Swift. This book is for those who already have programming knowledge. It's for you if you want to become a _Swift_ developer and learn the foundational principles.

In other words: if you want to add this new language to your tool-belt, or if you want to apply for modern iOS and Mac jobs, then you should read this book and keep it in arm's reach.

**[Buy _The Swift Developer's Cookbook_ from amazon][cb]**

If you want to take the next step, [Advanced Swift](https://gumroad.com/a/642135155/NcqF) goes very deep.


[arr]: http://ericasadun.com/2015/06/01/swift-safe-array-indexing-my-favorite-thing-of-the-new-week/
[cb]: http://www.amazon.com/Developers-Cookbook-Content-Program-Library/dp/0134395263/ref=as_li_ss_tl?ie=UTF8&qid=1459235590&sr=8-1&keywords=swift+developers+cookbook&linkCode=ll1&tag=chritietwork-20&linkId=35655d685eb281d01543bf74c48ffad9

