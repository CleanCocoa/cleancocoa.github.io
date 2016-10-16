---
title: Import Parts of a Module
created_at: 2016-02-26 08:51:00 +0100
kind: worklog
tags: [ swift, module ]
comments: on
preview: fulltext
---

Did you know you can import only parts of a module? Found this in the [Swiftz](https://github.com/typelift/Swiftz) sample codes:

    #!swift
    let xs = [1, 2, 0, 3, 4]
    
    import protocol Swiftz.Semigroup
    import func Swiftz.sconcat
    import struct Swiftz.Min
    
    //: The least element of a list can be had with the Min Semigroup.
    let smallestElement = sconcat(Min(2), t: xs.map { Min($0) }).value() // 0


`import protocol` or even `import func` were very new to me. Handy!
