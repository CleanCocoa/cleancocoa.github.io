---
title: 1 Criterion to Determine if You Should Write Unit or UI Automation Tests
created_at: 2016-02-03 10:03:25 +0100
kind: worklog
tags: [ testing, unit-test, functional-test, confidence, client, ui-test ]
image: 201602031008_decision-t.jpg
vgwort: http://vg01.met.vgwort.de/na/8346af3e9a0946cf8098dbb543ef9954
comments: on
---

Xcode now offers two kinds of tests natively. You don't have to rely on 3rd party JavaScript libraries anymore to automate the iOS simulator and assert view conditions. So with the advent of native UI Automation tests, can you rely on them to verify your app works?

Here's the two sides of the 1 criterion you'll ever need to find out:

- Write unit tests for to get confidence in your code. They are for you, dear programming friend.
- Use UI testing to verify overall app behavior which is interesting for the client.

So the criterion is: **who do you want to soothe with the tests?**

If you're both client and programmer, then the boundaries blur, of course. The basic advice is: Don't rely on UI testing to verify that your code works; use it to see if you broke any over-arching feature of the app, including web service queries and view transitions.

Also, no, don't abandon unit tests. They are important to verify what your code does. UI tests cannot do that. I'm sorry, but you'll still have to write unit tests even though they are hard to get right.

{% include figure.md src="/assets/blog/2016/201602031008_decision.jpg" alt="decision" url="http://url.com" caption="Photo Credit: <a href=\"https://www.flickr.com/photos/14404303@N08/7165197092/\">Decision</a> by Tim Rizzo. License: <a href=\"201602031008_decision.jpg\">CC-BY</a>" %}

There're a lot of different concepts and words floating around the web to distinguish the new type of test. Some call them integrated end-to-end tests; some functional. The Cocoa community is mostly absent from such discussion and seems to be busy getting work done[^1]. So let's stick with the test template names provided by Xcode.

[^1]: Frankly, I don't really think so; they are probably not more productive, they're just taking a higher risk here. I am flabbergasted how few people talk about the important topic of code maintainability. Cocoa programmers are in business for decades. What do you do, guys? Rely on your luck and experience alone?

J. B. Rainsberger, who is not only a well-known coach for introducing Test-Driven Development to teams but also a very avid blogger, coined the phrase "[Integrated Tests are a Scam][scam]". And this struck some communities hard, especially some Rails people who used end-to-end tests that hit the database a lot.

With UI Automation tests, the same fallacy may hit us Cocoa people. End-to-end tests are useful to show that a feature is implemented -- a happy path from logging in to posting the first cat picture. It's just they don't help drive the design of your code because they don't _call_ your code, and they never exercise everything in the app. How could a UI tests verify that tapping "back" and then going forward again does not break at the 101st time? There's no way to verify every possible path the user takes.

It is possible to gain confidence in code seams doing the right thing. When a counter is incrementing somewhere and passed to the method `showCounterLabel`, try to pass to it these values: -1, 0, 1, 5, 100000. Easy as pie when you call code directly in unit tests. That's what they are made for: test client code to your "library", exercising the API directly, showing flaws in component design [like exposing partial functions][part] or having lots of internal state spread across the app.

So don't ditch unit tests only because UI automation tests are available and seem to do so much more with every single test. J. B. put it best in a [blog post][p]:

> When I refer to the integrated tests scam, I mean that we shouldn’t use integrated tests to check the basic correctness of our code. I’m talking only about programmer tests: tests designed to give the programmers confidence that their code does what they think they asked it to do. Some people call these “technical-facing tests” or “developer tests” or even “unit tests” (I never use “unit tests”, because that term causes its own confusion, but that’s another article). Whatever you call them, I mean the tests that only the programmers care about that help them feel confident that their code behaves the way they expect. There, I write as few integrated tests as possible.
>
> When working with customer tests, such as the kind we write as part of BDD or ATDD or STDD or just generally exploring features with customers, I’m not using those tests to check the correctness of the system, nor the health of the design; I’m using those tests to give the customer confidence that the features they’ve asked for are present in the system. I don’t expect design feedback from these tests, so I don’t worry about the integrated tests scam here. If your customer tests run from end to end, I don’t mind; although you should remember that not all customer tests need to run from end to end. James Shore once told me about “customer unit tests”, which I’ve also used, but that’s another article.

Integrated tests are a scam because you continuously have to throw more of them at the problems but will never see which part of your code causes trouble. Bad unit tests are no silver bullet, either, but good ones can help tremendously.

[part]: /posts/2016/02/partial-function/
[scam]: https://vimeo.com/80533536
[p]: http://blog.thecodewhisperer.com/2015/12/08/clearing-up-the-integrated-tests-scam/
