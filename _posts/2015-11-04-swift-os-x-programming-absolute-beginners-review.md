---
title: Swift OS X Programming for Absolute Beginners Review
created_at: 2015-11-04 11:36:38 +0100
kind: worklog
tags: [ review, swift, appkit ]
image: bookcover-t.jpg
comments: on
---

Recently on Google+, someone recommended Wallace Wang's _[Swift OS X Programming for Absolute Beginners][book]_. Well, I'm not a beginner anymore, but the book sounded fun, so I gave it a spin. And I'm quite impressed. That's why you read the review here.

In short, _Swift OS X Programming for Absolute Beginners_ (or SOXPAB as I would like to call it to save myself from typing that much) is the best programming book to teach the reader about user interface programming. I can't honestly judge how well this textbook will actually work for non-programmers. But that's because I can't fathom learning to code from a book anyway. If you know programming, or the Cocoa APIs already, this should work for you.

The thing that makes SOXPAB so great is the structure of the book and its chapters. Wallace Wang's guiding principle is this:

1. Introduce the relevance of a topic
2. Show how this is coded
3. Show how to use this in a controller object plus Interface Builder

I don't particularly enjoy reading how to make decisions with branches anymore (`if` and `switch` mostly). Most textbooks do a very bad job at keeping this stuff interesting. (What's interesting or fun about boolean operators and scoping variables? Especially once you've learned this in the past already?) But Wallace Wang is clever and continues the chapter with building a simple Cocoa AppKit app to _see_ code branching in a real-world application.

This is his main selling point, and it's a good one.

The book should stress the "OS X Programming for Absolute Beginners" instead of "Swift", but then again "Swift" sells better, I guess.


## Contents

Here's the chapter headings of the 540+ pages book:

1. Understanding Programming
2. Getting to Know Xcode
3. The Basics of Creating a Mac Program
4. Getting Help
5. Learning Swift
6. Manipulating Numbers and Strings
7. Making Decisions with Branches
8. Repeating Code with Loops
9. Arrays and Dictionaries
10. Tuples, Sets, and Structures
11. Creating Classes and Objects
12. Inheritance, Polymorphism, and Extending Classes
13. Creating a User Interface
14. Working with Views and Storyboards
15. Choosing Commands with Buttons
16. Making Choices with Radio Buttons, Check Boxes, Date Pickers, and Sliders
17. Using Text with Labels, Text Fields, and Combo Boxes
18. Using Alerts and Panels
19. Creating Pull-Down Menus
20. Protocol-Oriented Programming
21. Defensive Programming
22. Simplifying User Interface Design
23. Debugging Your Programs
24. Planning a Program before and After Coding

The first 6 chapters are instructive and probably serve as a reference later on if the still new concepts are put to use. As I said above, in chapter 7 Wallace Wang already shows how to create a basic user interface and make it work with branches, loops, etc.

I wouldn't have thought that section headings like "Using Dictionaries in an OS X Program" exist, but there you are, every chapter from 7 to 12 applies basic programming principles to user interface programming. And with each example you'll repeat how to set up a Cocoa app and wire the widgets to their controller object.

Chapters 20 and 21 are probably rather quick additions for Swift 2 since the book was released around or even before Swift 2's public release. Doesn't do them any harm, though. As of today, the sample code is still valid.

Also, the last chapter is a bit of a disappointment: instead of focusing on traditional or agile principles of project planning and sketching, Wang says that it's simply up to you to decide which partition of a car works best: engine, gear, and wheels, or front, back, and top. Well of course it depends, but on what exactly he doesn't say: the domain you're modeling. Treating cars as physical objects among other objects will favor the latter structure. Interacting with cars as cars will benefit from an object that's called "engine" and knows how to accelerate.

In short, talk about object-oriented programming principles and design is a bit short and not very satisfactory. But there are [other books][des] for that anyway.

## My Recommendation

If you want to get started with OS X programming and would like to have a pragmatic guide which you can follow from beginning to end in a couple of evenings, this book might be for you. Getting to know Swift this way is great: Not too much abstract talking, lots of short samples, and getting practice with Cocoa/AppKit right away.

It's very cool to visualize inheritance with a simple demo app, for example. I think I'd have liked to learn Cocoa programming this way.

The book is both heavily illustrated and full of lists. I like lists because they're easy to follow. There's no point in writing high prose when all you need to convey is to perform 10 simple steps in a specific sequence. So it's an easy read, and if you follow through the examples, you should be done in a couple of evenings fiddling around with the code.

Since this book is aimed at "absolute beginners", you won't learn _why_ the author picks `NSAlert.beginWithCompletionHandler` instead of the old-fashioned `runModal`. There's not a lot of discussion and teaching of the intricacies. But as a beginner, you probably wouldn't want to know anyway.

But what _do_ you like to know? Best practices presented as the de facto standard are a good starting point. How will you learn to compare the only solution you know from a book with the plethora of different approaches you'll find on the web once things get hairy and you need new solutions? There should be an additional chapter about thinking as a programmer and teaching yourself new concepts, how to compare their utility and idiosyncratic drawbacks. But this is lacking from every book I know and not just Wang's fault.

I think Wallace Wang does a very good job at teaching the basics and then showing them to his readers in _Swift OS X Programming for Absolute Beginners_. That's a huge win.

Interested? [Buy it from this link from amazon][book] to support my writing -- amazon gives me a small kickback for referrals.

[des]: http://amzn.to/1SnJnbm
[book]: http://amzn.to/20tsGAB
