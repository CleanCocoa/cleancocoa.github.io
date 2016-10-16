---
title: Calendar Paste 2 is now Open Source
created_at: 2014-12-13 19:25:08 +0100
kind: worklog
tags: [ calendarpasteapp ]
shared_image: calendarpaste2appicon.png
comments: on
---

I have just pushed the current code of [Calendar Paste 2][calpasteapp] for iPhone to [GitHub][repo]. I did this in part because I don't fear anybody stealing it anyway, and also because I want to work on two machines this weekend without setting up my own server.

Two years ago, I [released][] Calendar Paste to the public. Initially, I scored a few weeks of very high sales volume. Then followed months in silence. With the [update for iOS&nbsp;7][cp2], Calendar Paste 2 managed to sell a lot better. Which isn't much, still (between 4 and 10 sales per week). It's not enough to make a living off. I don't care, and I will continue to push updates to the App Store -- because I think the app is useful, and because I learn so much from developing it.

In the upcoming weeks, I want to devote some time to making Calendar Paste 2 more useful: I want to add [Launch Center Pro][lcp] support, for example. I already fixed a bug yesterday where alarms couldn't be deleted.

The thing is, I have learned _a ton_ about application development in the meantime. I feel comfortable working in Apple's development eco-system, and I am able to design far more complex applications on my own now. I even wrote [a book on Mac application development][macappdev] recently. The code of Calendar Paste, though, didn't improve in the past years. Not a bit.

If you know something about programming for iOS, you will have noticed how bad the code is organized. It'll be a pain in the ass to add any feature _at all_, and my OmniFocus task list of nice-to-have things I want to do is huge!

Part of the problem is that everything is entangled. Theres `UIViewController`s responding to user input _and_ changing the domain model _and_ changing the user's calendar entry database. There's not a single automated test in place to catch problems if I refactor the code. Not one! Just because I didn't know how to do them in 2012!

In short, I discovered that the 2012-me left a bunch of legacy code behind.

Part of the exercise in the upcoming days to Christmas is to break the legacy code up into manageable pieces, and to put some order to the disastrous state it is in at the moment. The situation for budding iOS and Mac application developers has improved a lot, I think: there's a ton of good resources on the net, and StackOverflow is chock-full of good advice. Maybe I can help newcomers to learn from my mistakes and start learning on the right foot.

You may now have a look at the [Calendar Paste 2 GitHub repository][repo] and wonder how someone could by all means _ship_ an app like that. 

Then come back to my blog regularly to see how I improve it step by step.

[calpasteapp]: http://calendarpasteapp.com
[released]: /posts/2012/12/calendar-paste-release/
[repo]: https://github.com/DivineDominion/CalendarPaste-app
[lcp]: http://contrast.co/launch-center-pro/
[macappdev]: /posts/2014/12/exploring-mac-app-development-strategies-released/
[cp2]: /posts/2014/03/calendar-paste-ios7/
