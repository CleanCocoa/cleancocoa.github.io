---
title: Prototypes for Calendar Paste
created_at: 2013-04-24 23:31 +0200
kind: worklog
tags: [ calendarpasteapp, prototype, worklog, jquery ]
vgwort: http://vg02.met.vgwort.de/na/dd3a2247eef64bb2a83b209899e19b60
---

When Marko ([@markowenzel](https://twitter.com/markowenzel)) and I started to plan our first iPhone app codename "ShiftCal", we met and used prototypes to get a feeling for the Interface we wished Apple provided built into Calendar.app. The prototypes for the app which later would be dubbed "[Calendar Paste][cp]" were really fun to make.  

I can only recommend:  if you're going to create your own iPhone app, don't just dive into Xcode and Interface Builder.  When you use these programmer's tools for the first time, getting to know their components and technical hurdles will hinder your flow.

You won't have these issues with paper (prototypes).

  [cp]: http://calendarpasteapp.com

## Phase 1: Paper Prototype

{% include figure.md src="/assets/blog/2013/201304230937_paper.jpg" alt="Paper views" %}

We sketched around for some time and picked certain view layouts, dismissing ugly and clumsy ones.  That's an easy start, took us 15 minutes.

I think paper prototypes by themselves are seldomly fun to interact with.  You _can_ make some sort of live chinese paper theatre ripoff and switch paper snippets when your test user interacts with the prototype.

A non-interactive option is to shoot [stopmotion animations of your prototype](http://vimeo.com/6085753).[^ism]  With these, you can record and then present your interface interactions as replays.  That's not as revealing an experience like having real live testers, of course.

If you want feedback on the interaction, you should probably prefer interactive prototypes to replays.  No-brainer, isn't it?  Here's my minimal approach to that.

  [^ism]:  Software like [iStopMotion](http://www.istopmotion.com/) (no affiliate) seems to work for that stuff.  Currently on sale on [MacHeist](http://macheist.com/) until 2013-04-26.

{% include figure.md src="/assets/blog/2013/201304190930_keynote.png" alt="Keynote presentation" caption="Preparing the Keynote with slide links" %}

Since I always found it somewhat tedious to cut out interface elements as paper snippets and shuffle them around when the test user interacts with the prototype, I opted for a minimal technology-based solution.

Because paper alone didn't cut it, we took pictures of every view with our iPhones and determined which views were connected to one another in a little **flow chart**.  Marko then assembled a **Keynote presentation** with the appropriate dimensions and included the pictures I took.

{% include figure.md src="/assets/blog/2013/201304201355_js.gif" alt="interactive Keynote" caption="Animation of the interactive keynote" %}

Keynote (and PowerPoint, for that matter) provide ways to create hyperlink boxes on the slides.  This way we added interactivity to the Keynote prototype.  Still, there's a small caveat:  when you miss the hit targets in Keynote presentations, the presentation simply advances to the next slide.  There's no way to prevent this easily, though.

You know what?  We didn't see the opportunity to create a cool prototyping app like the folks at WOOMOO did:  they've released this truly useful app called [POP -- Prototyping on Paper][popapp].

Today I wholeheartedly recommend you use [POP][popapp] and ditch Keynote to create interactive paper-based prototypes.  It does what you want:  compose shots of your paper prototype into an interactive picture prototype.  You can even share it with others.  I like it.

  [popapp]: http://popapp.in/

## Phase 2: JavaScript Prototype

{% include figure.md src="/assets/blog/2013/201304230925_out.png" alt="JavaScript Prototype" caption="Stages of the jQuery-based interactive prototype. <a href=\"http://divinedominion.github.io/calpasteapp-prototype\">Use it.</a>" %}

In the last stage of prototyping, I went and created a jQuery/jQuery Mobile webapp.  It's now [hosted on GitHub](http://divinedominion.github.io/calpasteapp-prototype).  Try it for yourself!

Albeit it was a little more time consuming to get there than I expected, this prototype revealed a nice interface detail:  I found a way to get rid of all settings for Calendar Paste, which would consist of selecting a default calendar for new event templates.

Now the app asks the user whether his custom calendar selection should be stored as the default.  That's the only preferences entry I could think of anyway, so getting rid of that meant I wouldn't need to implement a "Settings" view at all.  Getting rid of that is always nice.[^caldef]

  [^caldef]: Since this is a webapp, I can hotlink to the [calendar picker view](http://divinedominion.github.io/calpasteapp-prototype/#calendar) so you can see for yourself how I imagined it would behave.  I like that I'm able to navigating the web app with anchor links.  Reminds me of custom URL Schemes on iOS.  Makes me think ...

So getting an interactive prototype ready to roll took some time.  But during the process I was able to think differently about the app's design.  Paper alone didn't induce such ideas.  Maybe they'd have come if I only spent more time with them, who knows.

1. *Bonus*:  I even ran the JavaScript prototype on my iPhone.  It wasn't very responsive I have to admit, but I nevertheless was able to test a few design assumptions on the device itself and took notes.

2. *Bonus bonus*:  this was the moment I wanted to reach out to other people and get their opinion real hard.  (I didn't.  _Chicken._)  This already is kind of a **[Minimum Viable Product][mvp] (MVP)**, and it's more than just a video presentation[^db], so why not get some real feedback from the community?

   In this stage the prototype is still a user interface test only.  For a "real" MVP it'd be nice to have it do _something_, like, say, create events into the real Calendar thing.  That's not allowed on iOS, though.

3. Here's another bonus tip oncerning web apps on the iPhone:  just [like the folks of Forecast.io said][fcio], you shouldn't imitate the native look and feel of an iPhone when you create web apps.  (I did and was ennerved by the delays.)  There are certain assumptions concerning the looks and behavior of native elements which you better not try to replicate (because you will fail).  Instead of leaving your users in [Uncanny Valley][uv], just do something different with your interface altogether and you'll be fine.

  [mvp]: http://en.wikipedia.org/wiki/Minimum_viable_product
  [^db]:  Did you know [Dropbox started out as a video-MVP](http://techcrunch.com/2011/10/19/dropbox-minimal-viable-product/)?
  
  [fcio]: http://blog.forecast.io/its-not-a-web-app-its-an-app-you-install-from-the-web/
  [uv]: http://en.wikipedia.org/wiki/Uncanny_valley
  

## Conclusion

Prototyping helped me get a feeling for the actual app.

I'm not very good at just juggling thoughts in my head.  Taking notes, sketching stuff, making Mind Maps---all that facilitates the process of thinking for me.  When I created these prototypes, I also created opportunities for rapid, visual and tactile feedback and learned stuff quickly by shortening the delay between user interface iterations.

Having Storyboards and the like in Xcode's _Interface Builder_ might do the trick for you, too, if you setup basic navigation and sketch a first half-functional UI draft.  When it comes to visually arranging the components by clicking it might even be a lot more fun than dealing with jQuery Mobile in code.

As [Florian Kugler pointed out][fk], ditching Interface Builder altogether might be worth it.  I didn't touch it once during development of Calendar Paste.[^fs]  But that's another chapter;  Interface Builder might be just what you need to get you started and prototype your app.  Just don't rule out the choice to create the final app in pure code later on just because you had a first interface draft prepared in Xcode's Interface Builder already.

Next time I'm going to use POP.  Since it's way more comfortable to prepare and use than Keynote presentations, it may even be sufficiently interactive so I won't need a closer-to-the-real-thing JavaScript prototype.

---

*See the [Calendar Paste development series overview](http://ct.dev/posts/2013/calendar-paste-development-series/) for more posts like this one.*

  [fk]: http://www.floriankugler.com/blog/2013/4/15/interface-builder-ndash-curse-or-convenience
  [^fs]:  *Spoiler alert:*  That's not entirely true.  I used a table view in Interface Builder to see how it'd work.  Hey, some of Apple's example codes use `.nib` files, so what else was I supposed to do?
