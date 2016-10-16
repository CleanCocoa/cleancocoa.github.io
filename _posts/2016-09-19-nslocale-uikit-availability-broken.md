---
title: Swift 2.3 NSLocale UIKit API Changes Break iOS 8 Compatibility
created_at: 2016-09-19 15:48:48 +0200
kind: worklog
tags: [ uikit, swift, radar ]
comments: on
---

We're converting a rather big and clumsy project to Swift 2.3 at the moment. Strangely, `NSLocale` gives us a lot of headaches. The API hasn't changed visibly, but the implementation seems to have.

From Objective-C's point of view:

    #!objc
    @property (readonly, copy) NSString *currencySymbol 
    API_AVAILABLE(macosx(10.12), ios(10.0), watchos(3.0), tvos(10.0));

And from Swift's:

    #!swift
    extension NSLocale {
        // ...
        @available(iOS 10.0, *)
        public var currencySymbol: String { get }
        // ...
    }

That property certainly isn't new. Neither are the dozen others in that header file from Foundation.

[iOS 10 Foundation API diffs](https://developer.apple.com/library/content/releasenotes/General/iOS10APIDiffs/Swift/Foundation.html) reveal:

> Added NSLocale.currencyCode\\
> Added NSLocale.currencySymbol

Huh? But they were there all the time! The app used these properties since iOS 8. It is clear something is wrong here. The compiler forces us to now access these properties with availability checks for iOS 10 and newer. The `else` clauses (for <10.0) don't know nothing about these properties at all, though. It's a compile-time constraint for a runtime issue (kind of).

Shadow protocols don't work:

    #!swift
    protocol Swift22LocaleFallback {
        var currencySymbol: String { get }
        var currencyCode: String? { get }
        var localeIdentifier: String { get }
    }

    extension NSLocale: Swift22LocaleFallback { }

The result is another compiler error, this time in Foundation.NSLocale, "Protocol 'Swift22LocaleFallback' requires 'currencySymbol' to be available on iOS 8.0.0 and newer". No chance.

You know what _does_ work? `performSelector`.

    #!swift
    if #available(iOS 10.0, *) {
        formatter.currencySymbol = locale.currencySymbol
    } else {
        let result = locale.performSelector(Selector("currencySymbol"))
        formatter.currencySymbol = result.takeRetainedValue() as! String
    }

This is a strong indicator for me to upgrade other projects to Swift 3 instead of 2.3. `#radarorgtfo` [rdar://28363526](http://openradar.appspot.com/28363526)
