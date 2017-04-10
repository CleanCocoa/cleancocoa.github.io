---
title: FatSidebar View Component for macOS Released
created_at: 2017-04-09 15:42:05 +0200
kind: worklog
tags: [ opensource ]
image: fatsidebar.png
url: https://github.com/CleanCocoa/FatSidebar
comments: on
---


One important user interface part of my latest top-secret project involves a sidebar of buttons. Like a regular toolbar, but taking up less space for chrome, looking more flat, and the user should be able to create toolbar buttons herself.

So while I was mostly sick at home for the last couple weeks, I spent my time cobbling this together. With drag and drop reordering and all.

{% include figure.md src="/assets/blog/2017/fatsidebar.png" alt="logo" url="https://github.com/CleanCocoa/FatSidebar" %}

**[Find `FatSadebar` on GitHub!](https://github.com/CleanCocoa/FatSidebar)**

I have never written my own custom view component from scratch before. I helped improve [KPCTabsControl][kpc] for Swift 3 when I created TableFlip last year. And of course I participated in a lot of smaller open source projects, too. But I never started from scratch, and that was cool.

Also fun: creating the library's own "logo". Made it feel so much more official.

What's cool about writing a new thing from nothingness is that I had no clue what to do and how to start. This component turned out as rather adventurous mental gymnastics because I had to leave the paths of application development I know so well. I still don't know all the answers; what are best practices? Get something colorful on screen? Customize `drawRect` and draw boxes and placeholders? Partition the view into sub-components using Auto Layout from the get go? Is drawing text better handled by `NSTextField` labels than `NSAttributedString.draw(in:)` or is the overhead too much? (I still don't know the best answer for this.)

Anyway! I ended up putting this together as a library with sample app. There are some unit tests for inserting items into the "fat sidebar", but otherwise I find the drawing and layout related code to be absolutely hideous. Cannot come up with improvements on that front that go beyond cosmetics, though. Maybe later, with more experience.

[kpc]: https://github.com/onekiloparsec/KPCTabsControl
