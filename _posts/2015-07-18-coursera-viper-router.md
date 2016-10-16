---
title: How Coursera Uses VIPER to Architect Transitions Between App Features
created_at: 2015-07-18 14:41:03 +0200
kind: worklog
tags: [ viper, software-architecture, router ]
image: 201507181442_viewcomponent.png
comments: on
---

Oy vey, now there's a ton of AltConf 2015 talks online, too! So much to digest! In ["250 Days Shipping With Swift and VIPER"][talk] by Brice Pollock, he shows how the folks at Coursera do some clever things with the [VIPER iOS app architecture][viper]: they use a `Router` which handles internal URLs.

These URLs are strings captured in an enum. Each routable component responds to routing requests with a success or failure indicator.

The routable components are called "View Compontents". A View Component, according to Brice, is made up of a View (a protocol implemented by the `UIViewController`), a [View Model][vm], a Presenter (and Event Handler), and an Interactor. Some of these terms are taken from [VIPER][viper].

A feature of an app, like logging in or displaying stats, is probably a combination of multiple View Components, but that's just a guess. Don't confuse this concept of uppercase View Components with actual view components (lowercase, see?), which are `UILabel`s and the like.

At about 10:50, Brice shows how events within a View Component are handled. In case you haven't tried VIPER, yet, that part should be pretty instructive.

So instead of transitioning from one `UIViewController` to the next, you would model a transition from View Component A to B, including switching Views or view controllers.

As I said [a while ago][segues]: an architecture like this doesn't work well with Apple's built-in mechanisms, like Storyboard Segues. I guess there's room for a VIPER-inspired app template to reduce architecture overhead per app. A framework won't be a good fit because there are so many places where you will want to customize how things are done.

Watch [his presentation][talk], it's worth it!

[talk]: https://realm.io/news/altconf-brice-pollock-250-days-shipping-with-swift-and-viper/
[vm]: /posts/2015/06/start-mvvm/
[viper]: /posts/tags/viper/
[segues]: /posts/2015/01/segues-vs-tell-dont-ask/
