---
title: Magic Numbers Represent Concepts
created_at: 2016-09-25 10:15:32 +0200
kind: worklog
tags: [ refactoring, clean-code, yourdon1979sd ]
comments: on
lang: mmd
---

Literals _represent_ something.[p 94][#yourdon1979sd] Magic numbers are a form of literal.

The number 79 can truly represent the result of 80-1. 

When `height = 80` and `offset = 1`, the resulting number 79 represents a relation between the height and an offset.

Working with "magic numbers", that is numbers that are not named using variables/constants in the first place, poses a problem when constraints or relations change. You cannot reliably "Search & Replace" every occurence of `79` in your project with `height - offset` -- some 79's could have meant something different before.

Thinking about readability first when you write code reduces these problems to a minimum later. Give literals meaningful names.

Swift accidentally provides **namespaces** for stuff like this with enums without cases. Then you don't clutter your functions with constant  definitions. The following example is quite convoluted, but brings the point across:

    #!swift
    enum Layout {
        static let height = 80
        static let width = 100
        static let padding = 1
        static let margin = (top: 0, right: 10, bottom: 0, left: 10)
    }
    
    func putStuffOnScreen() {
        let previousFrame: CGRect = // ...
        // ... 
        let newFrame = CGRect(
            x: previousFrame.x + previousFrame.width + Layout.margin.left,
            y: previousFrame.y + Layout.margin.top, 
            width: Layout.width + 2 * Layout.padding,
            height: Layout.height + 2 * Layout.padding)
        // ...
    }

[#yourdon1979sd]: Edward Yourdon and Larry L. Constantine (1979):  _[Structured design](http://amzn.to/2cm7ysC)_, Englewood Cliffs, N.J.: Prentice-Hall.
