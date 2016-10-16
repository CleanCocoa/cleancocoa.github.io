---
title: Tests Are Just Code, Too
created_at: 2016-09-09 07:40:39 +0200
kind: worklog
tags: [ tdd, testing ]
url: https://eev.ee/blog/2016/08/22/testing-for-people-who-hate-testing/
comments: on
---


From ["Testing, for people who hate testing"](https://eev.ee/blog/2016/08/22/testing-for-people-who-hate-testing/):

> The thing is, tests are just code. If you have a hard time constructing your own objects with some particular state, it might be a sign that your API is hard to use!

This is all there is to the magic of test-driven development, we could say. A test is _client code_. This means it doesn't have the inside perspective of the object or module under test. It has an outside perspective. Through tests you see how your production code is used (by other parts of your code or by other people in the case of libraries, say). 

You start with a test. This means: you start with an outside perspective, asking yourself questions like "What would be a good (public) interface for this?" -- The opposite is ad hoc reasoning, something along the lines of "I have this NetworkManager there and maybe just call it and return the JSON. Okay. Oh, now JSON doesn't work, I need to send a custom object along, so I'll just parse the JSON and create an object. But if that fails, hmm, I cannot send nil, so I'll force unwrap and let someone else deal with this." Or something. 
These improvised solutions can cause trouble because all they do is focus on the perceived requirements of the object you're writing, ignoring the requirements of code that uses the resulting code. 

Writing tests first kind of equals writing your app outside-in, starting with the calling code, then implementing the code that's to be called. Only test cases are super focused and you verify a lot more behavior and setup code with a single run than you could ever do with manual testing.

Experienced programmers will be able to switch perspectives even without writing tests. Their experience helps them to make it less likely to produce waste, whereas "waste" is code you write and then discard because you find it doesn't fit.
