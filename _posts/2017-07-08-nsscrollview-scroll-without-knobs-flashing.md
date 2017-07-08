---
title: Scroll NSScrollView Programmatically Without Showing the Scroller Knobs
created_at: 2017-07-08 09:47:10 +0200
tags: [ nsscrollview ]
comments: on
---

I am currently working on a typewriter mode text view. Even though this is a very popular trend for a couple of years, I couldn't find any open source component for this feature. I assume those who figure this out keep it a secret. If that is the case, it doesn't make that much sense nowadays anymore since every 3rd note taking app or so has such a feature already.

My naive implementation attempts got better over the past days. But keeping the line the user is typing in at the same position relative to the window all the time involves performing a scroll every now and then. But then the scroller knobs show. I tried to circumvent this for a while now, with subclasses and by overriding private API. Here's what ended up working:

```swift
class TypewriterTextView: NSTextView {
    // ... all the really interesting stuff ...
    func typewriterScroll(to point: NSPoint) {
        self.enclosingScrollView?.contentView.bounds.origin = point
    }
}
```

Call `typewriterScroll(to:)` instead of `NSView.scroll(_:)` to change the scrolled position without flashing the scroller knobs.

Up until a few minutes ago, I would've vowed that I tried this and it didn't work a few days ago. But it does work. Go figure.

Well, _power to the people!_ Guess I can show you a functional example, soon, with the rest of the component explained.
