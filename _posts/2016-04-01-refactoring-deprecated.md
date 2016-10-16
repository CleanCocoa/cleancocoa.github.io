---
title: Use Xcode Deprecation Warnings to Refactor
created_at: 2016-04-01 16:42:33 +0200
kind: worklog
tags: [ refactoring ]
comments: on
---

A refactoring in a side project requires finding places where a property we want to get rid of is currently used.

Finding the property name in the project will not work very well because other types use a similar named property. We only want to remove `Banana.size`, but not `Jeans.size`.

Deprecations to the rescue:

    #!swift
    class Banana {
        
        @available(*, deprecated=1.0)
        let size: Int
    }

It's even possible to provide instructions to co-workers:

    #!swift
    @available(*, deprecated=1.0, message="Replace with the new Canana")
    class Banana { ... }
    
Now Xcode will point out where we have to go to get rid of things and change logic. We can have a look at all of the places and perform changes so the existing code does compile (and test!) at all times -- only when no deprecation warnings show up anymore will we remove the property.

{% include figure.md src="/assets/blog/2016/201604011705_warnings.png" alt="Xcode warnings" caption="Deprecation warnings in Xcode" %}

The benefit of being able to compile at all times during the process is huge compared to removing the offending stuff right now and fixing compiler errors later.
