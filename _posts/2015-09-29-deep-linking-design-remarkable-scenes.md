---
title: "Universal Deep Linking on iOS: Design Each Scene to Be Remarkable"
created_at: 2015-09-29 09:08:27 +0200
kind: worklog
tags: [ ios, design, usability ]
vgwort: http://vg08.met.vgwort.de/na/207d47c0b2784781b3dfb6b477edc880
comments: on
---

Deep linking in iOS is all the hype. It's interoperability [opens up the platform](http://samanthabielefeld.com/journal/universal-deep-linking-in-ios-9) both for developers and users. But it brings new transitions to the platform and with them new responsibilities to design your application.

You can invoke scenes in other apps directly if they provide this functionality. Or you can route web links to in-app-links should you enable this feature.

Raluca Budiu of Nielsen Norman Group [points out][nng] that user will probably be very confused about their location if they are not too tech-savvy:

> Until recently, we used to say that apps don’t need a logo because users usually know what they’ve launched. If you expect your app to be invoked by other apps, **seriously consider adding a logo** in the navigation bar even on deep pages to tell people where they are. Otherwise, they may be not notice that they were switched to a new app and be confused when they don’t see the content that they expect.

You are not the measure. I'm a developer and I am able to keep track of context when using my device. I can anticipate a lot more of the platform's functionality than, say, my grandma. It's hard to do serious usability testing as a solo indie dev. But everyone of us can ask a friend or two to perform a task without knowing the application. Things like that will help.

Thankfully, usability researchers exist so we developers don't assume users are too much like ourselves. Their guidelines reduce developer's uncertainty.

The morale is this: **make your app stand out visually.** Transitions between apps require each scene of an app to provide sufficient context so users can identify it. Launching an app using its icon is straightforward. Being put in an app without knowing that this is possible is something different entirely.

I find adding a logo to the navigation bar weird, especially if you have none. Well-known brands will benefit, of course. But the rest of us have to focus on more than just the navigation bar.

Make it so users know what they look at when they see a screenshot of the app. Raluca will probably say the navigation bar alone should reveal enough context, but as a matter of fact, the user will look at the whole screen. 

**Design each scene to be remarkable.** Use the whole screen real estate to your advantage.

A new shade of blue as navigation bar tint won't do the trick if you don't have a logo. But even a logo in the nav bar won't make a crappy app look good. Above all, do something unique so users recognize where they are. Make your boring lists stand out. If everything looks like the built-in Settings app, well, then up your game a bit. Change some default colors, add subtle animations, and play with whitespace and combinations of font configurations a bit.

[nng]: http://www.nngroup.com/articles/ios-9-back-to-app-button/
