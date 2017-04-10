---
title: Shopify's Ruby on Rails app statistics
created_at: 2017-01-27 14:32:22 +0100
kind: worklog
tags: [ swift, ruby, statistics, oop, refactoring ]
url: http://web.archive.org/web/20170126202449/https://twitter.com/tobi/status/819246328211443713
comments: on
---


Shopify [published](http://web.archive.org/web/20170126202449/https://twitter.com/tobi/status/819246328211443713) amazing/epic stats for their Ruby on Rails "app":

* 860k LoC
* 1:1.3 code--test ratio
* 2300 model classes
* most classes <10 methods
* most methods way <10 lines

Makes you wonder:

* can you live with more types?
* can you shrink down existing types?

Swift changed the game a bit. Now it's very convenient to create a new type. You don't need to do the `.h`/`.m` file dance anymore but can start a new type declaration wherever you are. (Except when you're inside a generic type.) That makes it more likely we devs stop to shy away from smaller types.

But it's still a job for our community's culture to unlearn coding huge classes (massive view controller syndrome, anyone?) and splitting stuff.

Ruby has a long tradition of super-focused methods and very small classes. Swift is just as terse and powerful in that regard. Now it's your turn to experiment with writing types for very simple tasks. Like the [Extract Parameter Object](https://refactoring.com/catalog/introduceParameterObject.html) refactoring, where you lift things that go together into a new type.

Can be as easy as writing:

```swift 
struct DateRange {
    let start: Date
    let end: Date
}
```

_Et voilà_, there you have a new explicit concept in your code base.
