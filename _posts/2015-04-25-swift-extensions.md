---
title: Swift Extensions and Information Hiding
created_at: 2015-04-25 14:15:07 +0200
kind: worklog
tags: [ swift ]
url: http://www.andrewcbancroft.com/2015/04/22/3-nuances-of-swift-extensions
comments: on
preview: fulltext
---

Andrew Bancroft [tested and analyzed the behavior of extensions in Swift](http://www.andrewcbancroft.com/2015/04/22/3-nuances-of-swift-extensions).
While his findings aren't utterly surprising, having them summed up in this nicely done article certainly helps.

In a nutshell:

* extensions in the same file as their source can access even `privat` members (attributes and methods, that is)
* extensions in different files can access `internal` and `public` members -- but only if they are in the same module
* if extensions are in a different module, they merely have access to `public` members
* classes with `public` or `internal` visibility on themselves expose their members as `internal` by default
* classes with `private` visibility on themselves expose their members as private
* extensions themselves don't have to be `public` for `public` members to work (unlike classes)

This has consequences for your tests: they reside in a separate module, so they can only access `public` members of your classes.

If you extend classes from static libraries, the same holds true. Different module, only `public` accessors.
