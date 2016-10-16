---
title: Crafting Wicked Domain Models by Jimmy Bogard
created_at: 2015-02-05 17:00:32 +0100
kind: worklog
tags: [ domain-model, refactoring ]
image: 201502051701_ugly.jpg
comments: on
---


When we create applications and the business logic becomes more complex, it might be a good idea to focus on moving business logic into the Domain Model intentionally. If you don't do this, business logic will likely bleed into view controllers. Good luck finding the scattered remains when you need to perform changes!

A Domain Model is characterized by well-named classes which hold data and provide behavior. When you do something _to_ the data according to specific rules from your controllers, you are actually missing the opportunity to create a self-serving Domain Model. Instead, you mutate data containers from outside. The difference between a Domain Model and data container which merely wrap the persistence layer is the behavior of a Domain Model.

The Norwegian Developer Conference hosted Jimmy Bogard in 2012 for his presentation "[Crafting Wicked Domain Models][pres]". It's a great 1-hour video. Jimmy demonstrates how assigning offers to members of a coupon business became a one-liner in the actual service object. He got rid of all business logic in the service object. The process is worth examining. Jimmy presents a lot of great refactorings along the way which might get you inspired.

[Watch his presentation on Vimeo.][pres]

{% include figure.md src="/assets/blog/2015/201502051701_ugly.jpg" alt="Video screenshot" url="http://vimeo.com/43598193" caption="The Service object before the refactoring: the switch statement alone contains valuable business logic. But there's even more hidden in that method." %}


[pres]: http://vimeo.com/43598193
[summary]: http://programmers.stackexchange.com/a/221238/89665
