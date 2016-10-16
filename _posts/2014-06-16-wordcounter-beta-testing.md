---
title: Developing a Word Counter for Mac – Apply as a Tester
created_at: 2014-06-16 09:41:48 +0200
kind: worklog
tags: [ wordcounter ]
image: 201406161006_tally-counter_t.jpg
comments: on
---

I [promised][tt] to tell you more about what I was doing in the past weeks.

{% include figure.md src="/assets/blog/2014/201406161905_wordcounter-panel.png" alt="the main window" caption="The beta look of the status bar window" %}

I am developing a Mac application. Actually, I work on it since December. 
The application I'm working on is a word counter for Mac.  I believe that [tracking the daily output][track] is useful to stay productive, and I found tallying words written each day is a real booster. Whenever I write, I'm _happy_ to see that the counter increases and that I meet my goals.

If you're into tracking your time and progress but need a motivational boost to finish your writing, you should read [_How to Write a Lot_ by Paul J. Silvia][wrilo].[^aff] In this book, talk about "finding your muse" and "unleashing your inner writer" is refreshingly absent. Paul tackles the tangible instead: how to stick to a writing schedule, and how to track larger writing projects.

In this spirit of pragmatic problem solving, I developed a word counter to feed my statistics. To build good and lasting routines, you have to track the effect of your measure, like writing at a fixed time in the morning.

The Word Counter is tracking how many words you write each day. It also tracks how your output is distributed across applications when you put it on your [whitelist][].

{% include figure.md src="/assets/blog/2014/201406161006_tally-counter_t.jpg" alt="hand tally counter" url="http://commons.wikimedia.org/wiki/File:Tally_Counters.jpg#mediaviewer/File:Tally_Counters.jpg" caption="The Word Counter app works just like a bunch of tally counters. Nothing you write will be recorded.  Picture by Wikimedia user <a href=\"http://commons.wikimedia.org/wiki/User:Wesha\">Wesha</a> <a href=\"http://creativecommons.org/licenses/by-sa/3.0/\" rel=\"license\">cc</a>" %}

Is it secure to use this app? 

It won't and can not spy on your passwords: the Word Counter only takes a look at the key you type and takes a tally when you complete a word. It doesn't store _what_ you wrote. So, yes of course, it's perfectly safe to run the Word Counter in the background. It's not a key logger, it's a word tally.

## Call for Testers

Back in January I was already looking for [early alpha testers][alpha]. Then, the app was nothing more than a little service application. Now, it's almost finished and getting ready for a final 1.0 release.

I'm still polishing the overall look and feel. I need some more testers with Mac OS X Mavericks for this purpose.

If you're interested, [drop me a mail](/about): `christian.tietze` at `gmail.com`.  Beta testers get a big discount when the app is ready for sale, of course.


[tt]: /posts/2014/06/tapping-test-app-released/
[alpha]: /posts/2014/01/need-alpha-testers/
[track]: /posts/2014/02/count-your-words/
[wrilo]: http://www.amazon.com/gp/product/1591477433/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=1591477433&linkCode=as2&tag=chritietwork-20&linkId=2AGMZYIWAUSE36BM
[whitelist]: http://en.wikipedia.org/wiki/Whitelist

[^aff]: Affiliate link. You buy from this link, and I get a small kickback from amazon. No additional cost for you.

<!--ct: 
http://www.amazon.com/gp/product/1591477433/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=1591477433&linkCode=as2&tag=christietz-20

http://www.amazon.de/gp/product/1591477433/ref=as_li_ss_tl?ie=UTF8&camp=1638&creative=19454&creativeASIN=1591477433&linkCode=as2&tag=divined-21
-->
