---
title: "How to Get Started with Swift: Write New Tests in Swift"
created_at: 2016-05-08 10:55:16 +0200
kind: worklog
tags: [ testing, swift ]
url: http://indiestack.com/2016/04/test-with-swift/
comments: on
---

Daniel Jalkut of MarsEdit fame is slowly getting up to speed with Swift. I have a few Swift-only projects already ([Move!](https://christiantietze.de/move-work-break/), [TableFlip](http://tableflipapp.com/) and secret in-progress ones). On top of that, every feature I add to the [Word Counter](http://wordcounterapp.com/) is a Swift module, too. The main code of the Word Counter is still in Objective-C, though. 

Integrating new types to the project using Swift is awkward. There're things you can only use in one direction. Most if not all Objective-C code can be made available for Swift and works great. The other way around, not so much. Also, Objective-C imports from the `-Swift.h` file won't be available in Swift-based test targets. So you'll have to isolate these even further, create wrappers, or whatever. Porting a class to Swift can work, but the side effects on testing make it harder.

I like Daniel's advice to [write your unit tests in Swift](http://indiestack.com/2016/04/test-with-swift/) if you're still not certain if you should make the switch to this new and volative language. It's a good idea to get your feet wet.

Considering the problems I ran into with the Word Counter, though, I wouldn't know how, honestly.
