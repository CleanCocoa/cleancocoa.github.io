---
title: Global State Was Preferred Because Typing Sucks
created_at: 2016-09-03 10:07:18 +0200
kind: worklog
tags: [ programming, yourdon1979sd ]
lang: mmd
comments: on
---

According to Ed Yourdon, in the 1970's global state was actually widely preferred over passing parameters to functions or subroutines.[79--80][#yourdon1979sd] I had to laugh when I read the paragraphs on the most common objections, but this seems to have been a serious issue.

The objections listed are:

1. Typing parameters means more typing, i.e. more work
2. Getting the parameter list right is error-prone
3. Accessing global state used to be more efficient

Yourdon dissipates the objections:

1. Typing more parameters takes a bit of time now, but hunting bugs due to global state takes lots of time later.
2. Errors in getting parameter lists rights are local and easily fixed; again, hunting bugs which appear because of global state involves way more detective work.
3. Even in '79, machine time was less expensive than "people time": hunting bugs isn't worth the premature optimization.

But think about this for a moment. How spoilt we are using Xcode's or any other IDE's auto-completion feature to actually tell us parameter lists when we need them. Having to remember parameter lists and getting the sequence of unnamed parameters right was a problem sooooooo severe, people rather used global variables instead.

Crazy. Today people still have to be meticulously taught when and why global state causes trouble. In object-oriented programming, Singletons are the reincarnation of global state.
 
[#yourdon1979sd]: Edward Yourdon and Larry L. Constantine (1979):  _[Structured design](http://amzn.to/2cm7ysC)_, Englewood Cliffs, N.J.: Prentice-Hall.
