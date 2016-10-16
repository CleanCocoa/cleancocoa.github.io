---
title: The Power of Guard
created_at: 2016-04-12 08:49:42 +0200
kind: worklog
tags: [ swift, control-flow, guard ]
url: https://appventure.me/2016/03/29/three-tips-for-clean-swift-code
comments: on
---

Here's a guard statement with many different kinds of features [by Benedikt Terhechte][th]:

    #!swift
    guard let messageids = overview.headers["message-id"],
        messageid = messageids.first,
        case .MessageId(_, let msgid) = messageid
        where msgid == self.originalMessageID
        else { return print("Unknown Message-ID:", overview) }

I love `guard` statements. It helps make code much more readable and keep methods un-intended.

I feel less amorous about `guard-case` and `if-case` matching, though:

    #!swift
    if case .Fruit = foodType { ... }

This reads an awful lot like [Yoda conditions](https://en.wikipedia.org/wiki/Yoda_conditions) where you place the constant portion on the left side. This isn't even a typical boolean test (which would use the equality operator `==`) but a failable assignment (`=`). I can force myself to read and write it this way, but it always feels backward to me. It took me quite a while to memorize this, which I found surprising, as it was comparatively easy to learn Swift in the first place.

Apart from the `guard-case` portion, the example contains your standard `guard-let` unwrapping. The `else` clause is special though, and I didn't know this would work:

    #!swift
    guard someCondition else { return print("a problem") }

When the function is of return type `Void`, you can return a _call_ to another void function. This is shorter than splitting these two instructions on two lines, but it's too clever for every reader to understand. `print("a problem"); return` is just as compact, as Terhechte points out.


[th]: https://appventure.me/2016/03/29/three-tips-for-clean-swift-code
