---
title: Tips to Conquer Massive View Controllers
created_at: 2015-11-17 09:25:57 +0100
kind: worklog
tags: [ view-controller, refactoring ]
url: http://www.cimgf.com/2015/09/21/massive-view-controllers/
comments: on
---


View Controllers should only be responsible for view lifecycle events, [Marcus Zarra reminds us](http://www.cimgf.com/2015/09/21/massive-view-controllers/). That means they should populate views with data, and show and hide them -- stuff like that. View controllers should not do the work of views.

While I like his overall advice, it contains a few problematic points:

- Put Core Data out of the view controller. His solution to provide a single data source tied to a `NSManagedObjectContext` will not scale well [as I've experienced](/posts/2015/11/unit-of-work-context/). -- Still, don't tie it to the view controllers. That makes matters worse.
- His definition of "business logic" is odd. Business logic is not reducible to I/O code. It's not just about fetching data from a device and doing network requests -- except when your app is a very, very simplistic display for data with [CRUD][] operations. Business logic can exceed data-centric rules and entail a real domain with complex behavior without much imagination.
- Views should be responsible for displaying data, agreed. Since validation problems will be presented to the user, validation error messages will be part of that data, too. But views should not validate. [Validation rule objects][val] (WWDC'14 talk) should be the strategies that do validation.

> If we create an object to sit between the view and the view controller then we are creating unnecessary additional objects.

Where do [presenters](/posts/2015/10/view-model-control/) sit? Between view and and view controller -- but they make it easier to read and extend the code. They're hardly unnecessary.

I strongly believe that it's impossible to give advice about how to code and architect something if the problem domain isn't part of the example. Because everything depends on the problem space in the end. 

Remember to take every advice with a grain of salt. Collect things like Marcus' tips to obtain new tools, but don't take the tool for the solution of modelling a program to solve real problems.

[crud]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[val]: https://developer.apple.com/videos/play/wwdc2014-229/
