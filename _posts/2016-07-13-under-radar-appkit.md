---
title: "First Contact with AppKit for iOS Developers (Under the Radar #26)"
created_at: 2016-07-13 20:05:09 +0200
kind: worklog
tags: [ ios, mac ]
url: https://www.relay.fm/radar/26
comments: on
---

During my rather relaxed late evening workouts I catch up on podcasts now. Although I don't enjoy listening to people talk (I prefer reading), this is a great source of inspiration and seems to be the premier way in which people share their experience.

Marco Arment and David Smith run _Under the Radar_. In [episode 26](https://www.relay.fm/radar/26), they talk about Marco's deviations into AppKit---Mac app development. (Their shows are always under 30 minutes, so I suggest you give it a try.)

Marco's app [Quitter](https://marco.org/apps#quitter) is available for free. He couldn't put it into the Mac App Store because its main feature is to quit other apps. That doesn't work with Sandboxing. So Marco had to release it directly. 

I found a few things especially noteworthy:

* Direct distribution means you have to worry about installation, updates, and payments yourself. It's more work. So Marco made it free. (P.S.: [I wrote a book on direct distribution.](https://christiantietze.de/books/make-money-outside-mac-app-store-fastspring/))
* Marco was rather afraid to ship an update: maybe he didn't test thoroughly and introduced a bug; it seemed to me he missed the safety net of app reviews.
* Then again, shipping a bug fix update is fast and easy, too. It takes zero time to make an update available to customers.

The absence of app reviews in direct distribution is both frightening and liberating. I am afraid to ruin my customers's lives with every Word Counter update I ship. It's true, you can break things. But you can also make things right within the hour. You don't have to wait for anybody else. (Even though iOS [app review times](http://appreviewtimes.com/ios/annual-trend-graph) have improved a ton, from 7 to 2 days!)

I am astonished that both Marco and David have _years_ of experience working with UIKit but didn't ship a single Mac app in the meantime. Or do contract work back in the day. "iOS exclusive" sounds insane to me, but for them it has worked quite well!

**What's your experience? Do you remember your first contact with AppKit? Are you afraid to try?** Comment, email, or tweet me your response!
