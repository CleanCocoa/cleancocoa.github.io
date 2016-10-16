---
title: Making the Word Counter More Useful
created_at: 2014-10-24 10:20:33 +0200
kind: worklog
tags: [ wordcounter, nanowrimo ]
comments: on
---

Last week, I released an update which introduced a calendar view to the [Word Counter][wcapp] where users could look at their past achievements. I call this calendar view the History.

{% include figure.md src="/assets/blog/2014/201410241023_history.png" alt="Calendar View" caption="The Word Counter History view shows past achievements" %}

There's a lot of stuff on the roadmap still. The History is useful, but still pretty basic. I want to add analysis features. At the moment, users can look at the amount of words they write each day. But they have to find out if a day was only a good day or if it was an excellent day themselves. The Word Counter will soon help users with this, offering comparisons, calculating the mean, and helping users reach their daily goals.

To add analysis features will be pretty exciting, but I also know that it'll take a lot of time. Before I focus on analyzing data, I want to make the Word Counter track more kinds of data, first: the click-to-hit ratio to measure mouse usage; refine the word count algorithm; watch project progress.

I can't wait to implement this stuff because I know what we can _do_ with it. There are lots of possibilities to derive meaningful conclusions from the data. I'm focused on the technical side for now, but the user's benefit is something different than having just another count to look at. Users will finally get to know when and how they work.

The Word Counter is right now giving you feedback on your writing productivity. But it can't yet tell you what you may have to do. To achieve that, analysis features are necessary. That's the real deal. I'm building up momentum to reach that point soon. Still I'm amazed how long things take when you're indie and on your own.

Maybe it's because I pay too much attention to the architecture of my software, writing automated tests, rearranging pieces to fit my mental model when it changes. I know it pays off because I stay productive and the things still make sense to me. But it takes so much time, it's staggering.

So up next is the ability to monitor files and folders for changes and to count how many words users keep in their projects. Especially if you have to hand in an article which has to meet certain limitations, it's useful to know how well you fare, and when you procrastinate on the task. The evaluation may show if you unconsciously miss opportunities to invest in certain projects.

With [National Novel Writing Month][nanowrimo] just around the corner, I think I should rather hurry to make the Word Counter more useful to the budding writers out there.

You know, it's a good opportunity to [buy the Word Counter for Mac][buy] while it's still on discount until I finish file and folder monitoring.


[nanowrimo]: http://nanowrimo.org/
[wcapp]: http://wordcounterapp.com/
[buy]: https://sites.fastspring.com/christiantietze/instant/wordcounter?source=ctde
