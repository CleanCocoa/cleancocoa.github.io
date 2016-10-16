---
title: Breaking NSLocale API Changes Revisited
created_at: 2016-09-27 09:52:03 +0200
kind: worklog
tags: [ swift, uikit, radar ]
comments: on
---

Apple have responded to my issues with [broken `NSLocale` property availability](/posts/2016/09/nslocale-uikit-availability-broken/). The properties are actually convenience accessors which you can re-impement youself for use before iOS 10.

> These properties are new in iOS 10, and you could add equivalent properties for older OSes using an extension like the one shown below:

    #!swift
    extension NSLocale {
       var compatabilityCurrencySymbol: String? {
            return objectForKey(NSLocaleCurrencySymbol) as? String
       }
    }

Although that might work, the argument isn't sound: the properties were available before iOS 10. [rdar://28363526](http://openradar.appspot.com/28363526)
