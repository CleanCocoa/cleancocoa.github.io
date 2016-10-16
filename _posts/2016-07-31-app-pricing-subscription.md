---
title: "Ownership of the Apps You Use: Ulysses App Pricing, Subscription Models, and the Death of Licenses?"
created_at: 2016-07-31 20:59:12 +0200
kind: worklog
tags: [ pricing, subscription, indie ]
vgwort: http://vg01.met.vgwort.de/na/b45bb21e25094825895c143cbb942875
comments: on
---

It started innocently enough, with a customer being confused about paying for "the same app" twice. Now I wonder if the traditional pricing strategy for software is obsolete.

I found this on Twitter, and then I got hooked:

> @fehnman @ulyssesapp like I said, it's how you choose to sell it. +£50 for an app is pushing it.  
> ---@eatmorefish (9:35 AM - 31 Jul 2016)

Huh? I never considered Ulysses to be expensive. See [the full conversation on Twitter](https://twitter.com/eatmorefish/status/759653682594123776). 

> @ulyssesapp I bought Ulysses forgo Mac, but do I really have to buy again for my iPad? You sell syncing as a feature...  
> --[-@eatmorefish  (Jul 30)](https://twitter.com/eatmorefish/status/759416994902999040)

"Why, of course you do. Because buying an app for each platform is the way it works." I mean, that's how it always worked. It's software, duh. -- But _@eatmorefish_ had something different in mind:

> @fehnman @ulyssesapp it's how you choose to sell it. I don't pay extra for #adobe Creative Cloud. One price, all devices.  
> ---[@eatmorefish  (30 Jul)](https://twitter.com/eatmorefish/status/759511328960745473)

So user _@eatmorefish_ doesn't want to buy a license to own the right to run an executable file. (If you state it this clearly, it doesn't sound very appealing, does it?) Nope, he wants to purchase access to Ulysses **as a platform.**

Now take a moment and let this sink in.

---

With iOS and Android mobile devices, we now truly live in a cross-platform world. It's not PC or Mac anymore, or Windows or Linux. You have to factor mobile devices of any sort into the equation.

And, of course, people want to have their favorite apps on all devices. To them, it really is "the same app."

Nowadays you have to pay for "the same app" for every platform. Tech-savvy people understand. The rest isn't getting it. It's "the same app," after all!

Back in the times when cross-platform shareware was available for Windows & Mac, a single license key would unlock the app on both platforms. My friends at [Texts](http://www.texts.io) do it this way to this day, still. It's an honest approach. It's just that we developers cannot do this on iOS. 

The closest we get conceptually is making the app free, then add In-App Purchases for the killer features so there's a paywall to support your survival -- and then offer unlocking the features with license keys as an alternate mechanism. Only that I doubt Apple will allow that. It really is a dead end.

---

You know what's wrong with this thinking: Ulysses for Mac is not "the same app" than Ulysses for iPhone and iPad.

This thinking stems from the perspective of a layperson. As a Cocoa developer, you just know that making an iOS app is a totally different beast. You have to write nearly everything again from scratch. Or about 80% if you're lucky. (I just made that number up.) The platforms are similar, yet so different. If your code isn't designed to be truly Universal from the start, you're screwed. Ask [Literature & Latte](http://www.literatureandlatte.com) who made Scrivener for iOS why it took them so many years when they had access to the source code of the Mac _""version.""_ Ask [The Soulmen](http://www.ulyssesapp.com/team/) of Ulysses what took them so long. Ask anybody who ships "the same app" for both iOS and Mac. It is additional work, lots of it. Why do it for free? That doesn't make much sense from the perspective of a company!

But it does indeed make sense to offer a free iOS version from the perspective of customers. At least if they think about the Mac app as the real thing and the iOS app as an auxiliary means to access the data, for example. Or that in their mind both are the same and that you should pay $50 on the iOS App Store, too -- but then get in for free for Mac.

Lots of web services offer their apps for free to grant access to their platform with a native user experience. Like [Basecamp](https://basecamp.com) of former 37signals. Or LinkedIn. Or Facebook. 

How can they afford not to make money from their apps? 

Because they make money from their platforms.

The mobile apps become a tool to access the platform in a convenient way. It's rational to give them away for free to please users. It doesn't matter much if the platform is accessed via the web interface or the native app. As long as people are happy, the platform will thrive. It's a win--win situation to give these apps away for free.

---

Now Ulysses isn't a web service. Granted, it does sync your data over the web. So where's the difference to Microsoft Office or Adobe Creative Cloud? Well, Ulysses uses iCloud, so it's really Apple who offers the infrastructure. But do users really care about these implementation details? 

I guess to us developers it comes natural to think about Ulysses in terms of an offline app. From the fact that Ulysses is an app you buy, install, and then run locally somehow follows that you shouldn't pay for it on a monthly basis. It's just ... not how things work.

A subscription model [nearly ruined TextExpander](http://brettterpstra.com/2016/04/12/the-textexpander-subscription-snafu/), it seemed. -- "It's a productivity tool. I gladly paid for 2-yearly feature upgrades since 200X, but $40 per year? For what?!"

Could Ulysses the writing app become Ulysses the writing _platform?_ Could Ulysses make a monthly subscription model work?

That's very hard to tell. The backlash will be immense as long as our mental model is in alignment with "locally installed software." You purchase a license to use apps on your computer. It's personalized. It becomes yours. Like you buy a knife for your kitchen. Only it's digital.

Only when we switch our mental model from "locally installed software" to "cloud platform" will it be possible to justify a monthly fee. 

Microsoft Office seems to do well. Adobe Creative Cloud seems to do well. They offer cloud storage (and consequently spend time and effort maintaining their data centers) and they offer free updates for the binaries you download onto your computer. Stuff you do not download gets updated automatically, just like nobody notices Facebook has rolled an update until the "Like" button becomes a "Like" "palette", or whatever they call this now. 

For as long as you're subscribed to these services, you basically get access to the applications on your computer on a rental basis. This is different from software licenses where you buy the right to use the software once and can continue to use it for as long as your machine is capable of executing that file.

What I find the most interesting: that a user simply expected Ulysses to work this way already.

It's interesting that "an app" is not an executable file anymore, at least not in the mind of some people. "An app" has become a service. It means "access." You can use "the app" on any device you own, switching freely between them.

This kind of connectivity is just the next logical step towards moving consumer applications into the cloud. I do not endorse this kind of future. But I'd love to give consumers the ability to pay one price once and have access to everything from anywhere. Until version 2 comes out and they have to pay an upgrade price, heh.

---

Ideally, I would want people to ["patronize" me](http://blog.nextmarket.co/post/77108436218/on-patreon-and-the-new-patron-model-for-digital) for my development. I don't want to worry about selling stuff. I want to create great things first and not have to optimize for search engines, app store discovery, and the like "just to make it." Everybody needs money to survive, me included. So developers and writers, too, need money to create their stuff. But is the traditional model of purchasing licenses the best way to maximize utility of the product and increase revenue for the creators?

At least on mobile you cannot ask sustainable prices. You have to make apps cheap. Maybe because mobile phones didn't feel like serious computers. Doesn't matter much, anyway. It's just the way things roll now.

Think about the following scenario: Everyone gets my stuff for free. Download, use, enjoy. The Word Counter for Mac, TableFlip, Calendar Paste 2 for iOS, my books -- everything. I like that scenario because I love to create stuff. I really _want_ to give my creations to the world. 

Where's the fun in creating software when you hide it behind a paywall? 

Who learns anything from a book they cannot afford?

This would be just another kind of "open access."

Earlier this year, I gave away Calendar Paste 2 for free for 7 days. I scored 4000 downloads. (That's more than 4x the amount of purchases I had since its release in 2013.) That felt nice, even though I didn't get anything in return. Imagine a mere 1% of these people now use the app daily. That means I improved the lives of 40 people. Isn't that cool? The mere thought makes me happy. People use an application I created. How amazing is that?!

Marco Arment seems to do well with [patronage for Overcast](https://marco.org/2015/10/09/overcast2), judging from _Under the Radar_ episodes. At least [one person is worried](https://medium.com/@rmateu/overcast-2-and-the-burden-of-patronage-675c80411729#.mplmnejks) about patronage without an exclusive membership area or something else in return. Because patronage feels like giving a coin to a beggar on the street? I don't agree with that view.

Without patrons, most artists couldn't have painted their paintings or sculpted their sculptures. Crowd-sourcing patronage while doing art also is a logical next step in this highly connected world. (As opposed to moving everything into a centralized (!) cloud, I support that future.)

---

Here's a thought experiment:

Would you prefer to pay $30/month or so for every software you use _in total?_

Imagine you never had to pay for any app on an App Store to make this thought experiment work. Imagine you never spent all these hundreds or thousands of dollars for paid software. Downloads are completely free.

$30. Every month. You have access to your Ulysses, your Scrivener, your Word Counter, your Fantastical, your Keyboard Maestro, your tax program, everything. If you like to give more, maybe incentivized by the sheer amount of applications you use on a daily basis, you increase the amount to $50/month. Still not an awful lot. And if you're broke, then you give $5 or $10. (Every penny counts when the amount of payers is sufficiently large.)

You can download the app on all platforms, get basic cloud sync "for free" (premium plans would come at a higher cost), get regularly updates, and maybe even receive special membership perks for some of the apps you use.

In this world devoid of good old software licenses, how would you feel about patronage?

Would you enjoy this pricing model? 

Would you prefer it to the current model of "owning" a license? (What kind of ownership do you imply anyway?)
