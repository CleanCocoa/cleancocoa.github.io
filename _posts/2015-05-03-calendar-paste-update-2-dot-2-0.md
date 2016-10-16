---
title: Calendar Paste 2.2.0 Release Is Out
created_at: 2015-05-03 10:24:20 +0200
kind: worklog
tags: [ calendarpasteapp ]
shared_image: calendarpaste22appicon.png
comments: on
---

{% include figure.md src="/assets/blog/2015/201505031042_cpapp-framed.png" alt="Calendar Paste 2.2.0" caption="Calendar Paste looks good on all iPhones available now" %}

After working on it in my spare time for the last couple months, I have released a rather large update to [Calendar Paste][cpapp] this week.

* I wrapped [Cocoa singletons in my own](/posts/2014/12/refactoring-singletons-in-ios/),
* I introduced [Storyboard Segues](/posts/2015/01/segues-vs-tell-dont-ask/) but still think they're weird to use sometimes, and
* I use [success and error callbacks](/posts/2015/04/error-callbacks-objective-c/) instead of pattern matching a return value.
* Above all, I added unit tests by breaking up mangled code.

This update is mostly behind-the-scenes stuff. Visible to the user are a few color changes, and adapting iOS 8 looks better. The app looks nice on iPhones of all sizes available for the first time.

All thanks to making the switch to Interface Builder.

It enables me to work on an iPad version without breaking old stuff, because the old stuff is either gone or way more flexible than before. My old code was brittle -- and it hurt the eyes. The new code is way nicer to work with, although it can still use a refactoring or two (or ten).

Since day 1, people asked for an iPad version. I have written the first release completely by hand and found a universal release unfeasible. There were so many view frame calculations involved, I couldn't imagine doing this another time for a bigger display. I implemented all views in code. I didn't even touch Interface Builder because I didn't trust it. That was a mistake.

My key takeaway of the past 2 years with Calendar Paste is this: **use Interface Builder.** It's not like a Java visual editor from 10 years ago. It's the premier user interface design tool for Cocoa and CocoaTouch products.

I threw out so many lines of code, so many subclasses, and so much visual edge-case handling that I can't believe I really thought learning iOS this way was a good idea in the first place. It wasn't back then, and it isn't now. I only learned to hate view programming because it's so cumbersome to do in code.

Auto Layout helps a lot. Not having to build and run the app to see if a change looks good helps a lot. Modern Xcode tools are amazing, and you should use them in case you still hesitate. You don't learn any arcane tricks by programming all the views. It's just a waste of time until you really have to.

I'm happy I released Calendar Paste in 2012, even though I would feel ashamed to show the code to anybody. To release a product is magic. Updating it not so much. But every time I put out an update for any of my products, I am afraid I break something for somebody.

Would this change if I had a team to work with, people who can help catch errors?

Well, up next, I'd love to include a landscape view (especially for iPad) where you can drag and drop event templates to specific dates. It'd be way more intuitive than the boring date picker. That's a lot of things to learn to make this happen. I hope to do so this summer.

[Take a look at the website][cpapp] which matches the new (flat) look. Handling flat design is way easier than image-heavy looks.

[cpapp]: http://calendarpasteapp.com
