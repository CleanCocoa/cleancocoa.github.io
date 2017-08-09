---
title: Show a fat iOS-Style Insertion Point in NSTextView 
created_at: 2017-08-09 11:13:06 +0200
kind: worklog
tags: [ nstextview ]
comments: on
---

I have no clue why my previous attempts at customizing `drawInsertionPoint(in:color:turnedOn:)` always produced visual glitches. I really tried a lot of different ways. But it turns out you don't have to do that much, really:

```swift
class MyTextView: NSTextView {
    var caretSize: CGFloat = 4
    
    open override func drawInsertionPoint(in rect: NSRect, color: NSColor, turnedOn flag: Bool) {
        var rect = rect
        rect.size.width = caretSize
        super.drawInsertionPoint(in: rect, color: color, turnedOn: flag)
    }

    open override func setNeedsDisplay(_ rect: NSRect, avoidAdditionalLayout flag: Bool) {
        var rect = rect
        rect.size.width += caretSize - 1
        super.setNeedsDisplay(rect, avoidAdditionalLayout: flag)
    }
}
```

Adapted to Swift from [a Gist by koenbok](https://gist.github.com/koenbok/a1b8d942977f69ff102b).

That's all it takes. Marvelous!

**Update:** So it turns out that the drawing works just fine in a test project, but I end up with 1px-wide insertion points/carets/I-beams when I click with the mouse or press up/down arrow keys. Left and right work fine. I wonder what's wrong with this. Isn't it supposed to be a rather simple customization?

