---
title: "InfiniteCanvas – Vector Drawing App Concept"
created_at: 2017-01-15 07:22:22 +0100
kind: worklog
tags: [ infinitecanvas, concept ]
image: 201701150826_infinitecanvas.png
comments: on
---

I have no clue how you create drawing apps. I guess your primary concern is low latency and good-looking results rather than making the most of cool new architecture patterns like VIPER. It's a real-time thing. Not unlike games, I imagine. Still, "in between years" (between Christmas and New Year; but I like the literal German translation), I took a day or two to come up with [a conceptual implementation](https://github.com/CleanCocoa/InfiniteCanvas) (MIT licensed).

I use a [WACOM Intuous pen tablet](https://www.amazon.com/Wacom-CTL490DW-Digital-Drawing-Graphics/dp/B010LHRFM2/ref=as_li_ss_tl?ie=UTF8&qid=1484461838&sr=8-1&keywords=intuos&linkCode=ll1&tag=chritietwork-20&linkId=c1f19e11342bd51ab0a3643196b92a20) (affiliate link) instead of a trackpad or mouse most days. I fell in love with the quality of WACOM tablets some 10 years ago and never want to miss one of these things again for sketching ideas and drawing diagrams. With the digital pen, I can replace most of the scribbling I do to facilitate thinking when I'm programming. I create quick sketches of class hierarchies to _see_ where refactoring might help. I hand-draw collaboration diagrams and sequence diagrams to get a feeling for the runtime processing of data.

And now I want a slick app that can pan and zoom infinitely.

{% include figure.md src="/assets/blog/2017/201701150826_infinitecanvas.png" alt="prototype screenshot" url="https://github.com/CleanCocoa/InfiniteCanvas/" caption="Screenshot of a working but incomplete prototype" %}

So I set out to create it myself. Without any prior knowledge of how drawing applications work. (Hard to find a tutorial for that, too!)

My observations so far:

* AppKit's mouse events work well enough to draw smooth paths. While _recording_ every point that comes in does work, drawing a `NSBezierPath` through all of them slows down drawing pretty quickly.
* `mouseDown` means "start a new stroke", `mouseDragged` means "draw path through here", and `mouseUp` means "finish stroke". Drawing a stroke is pretty simple so far.
* Organizing the picture in terms of [`Stroke`s](https://github.com/CleanCocoa/InfiniteCanvas/blob/1afc97cbcc3be34dbd8da7bdc866dee531c1b632/DrawingTest/Stroke.swift) that are made of `Point`s and that expose (cached!) bounding rectangles helps speed up drawing: in the canvas view's `drawRect(_:)`, I only draw strokes that overlap the incoming `dirtyRect`.
* As an approximation, it suffices to add a point to a stroke [with a minimum distance of 4pt](https://github.com/CleanCocoa/InfiniteCanvas/blob/1afc97cbcc3be34dbd8da7bdc866dee531c1b632/DrawingTest/Stroke.swift#L37) to the last point. Ideally, the required distance would be smaller when you draw curves and larger when the line is very straight. I don't like freehand vector drawing tools that show a precise path while you draw, then "optimize" the path so it doesn't look at all like what you drew. (Looking at you, Inkscape!) But a clever algorithm can at least make stroke paths a bit more clever.
* Live-interpolation of paths as you draw them works for very long and complex paths at reasonable speed, but the CPU load is way too high. Instead of storing the points the cursor passed through, smooth the path once and store the Bézier-path triple of  `(point, controlPointA, controlPointB)`. That makes SVG import/export easier, too.

So you're very welcome to [contribute to the project](https://github.com/CleanCocoa/InfiniteCanvas) if you want to experiment with a simple drawing app! Once I got past the initial hurdles, all of the remaining steps seem to be very straight-forward. I created [GitHub issues](https://github.com/CleanCocoa/InfiniteCanvas/issues) for most of the things that need to get done already.

Until the app is ready, I'm using a cross-platform drawing app for artists, by the way, called [Mischief](https://www.madewithmischief.com). It's on the Mac App Store, too. It works pretty well for what I want to do, but it can only save the drawing as a rasterized image. Also, I don't like locking people into data silos and proprietary file formats. 

I'd like to make _InfiniteCanvas_  SVG-based for maximum compatibility with other vector drawing software. That's part of my general attitude that is best described as a "software-agnostic programming" approach, a monicker by my friend and [Zettelkasten](http://zettelkasten.de/) co-author Sascha. It means that good software should make itself irreplaceable through _quality_ while giving the user maximum power over the data. Locking people in by design is a sign of weakness. That's why plain text writing apps are morally superior. Same for SVG. Open formats, rule the world! ✊
