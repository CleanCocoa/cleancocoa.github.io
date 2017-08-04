---
title: How to Use NSGlyph in Swift
created_at: 2017-08-04 14:42:34 +0200
tags: [ swift ]
comments: on
---

I couldn't find a simple answer on the web at first, so here's my take for Googlers.

When you need `NSGlyph` (which is a `UInt32`), you probably want to use `NSATSTypesetter.insertGlyph(_:atGlyphIndex:characterIndex:)` or `NSGlyphStorage.insertGlyphs(_:length:forStartingGlyphAt:characterIndex:)` which in turn is implemented by `NSLayoutManager`. But the useful glyph types to use like `NSControlGlyph` are `Int`s. How do you get a `NSGlyph`-pointer from these?

Thankfully, in Swift you can satisfy `UnsafePointer<NSGlyph>` in two useful ways:
    
1. Pass a reference to a mutable variable with the `&` prefix, like:
    
        let glyphIndex = ...
        let charIndex = ...
        var glyph = NSGlyph(NSControlGlyph)
        layoutManager.insertGlyphs(&glyph, length: 1, forStartingGlyphAt: glyphIndex, characterIndex: charIndex)
2. Pass an _array_ of immutable values that is going to be kept alive for long enough, like:

        let glyphIndex = ...
        let charIndex = ...
        layoutManager.insertGlyphs([NSGlyph(NSControlGlyph)], length: 1, forStartingGlyphAt: glyphIndex, characterIndex: charIndex)

Since the exemplary `insertGlyphs` method call uses a plural-s, it's pretty straightforward to use an array once you stop worrying about the question "where do I get a pointer from?"
