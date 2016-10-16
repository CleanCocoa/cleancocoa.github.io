---
title: Conflicting Mental Models in TableFlip's Interface Design
created_at: 2016-07-02 07:40:03 +0200
kind: worklog
tags: [ tableflip, ux ]
comments: on
---

A beta tester of [TableFlip][] suggested I make the table grow automatically when one navigates to the edge of the table and presses the arrow key in direction of the edge. I am conflicted about this proposal.

**On one hand,** being able to grow the table easily is important. Spreadsheet applications usually present you with an infinite canvas. This is behavior people know (and maybe even expect).

**On the other hand,** a Markdown table is no spreadsheet. It's probably supposed to go into a document, be it HTML for a website or TeX for typesetting a book. An infinite canvas doesn't make much sense for that case's mental model, where you "draw" a table with boundaries intentionally.

Now the price question is this: does the mental model I had in mind matter? Is it a good one? Does it fit? This is the kind of stuff user testing can surface.

I imagine people will be surprised when the table grows only because they pressed an arrow key. But since I'm working on a [pruning][] feature anyway, removing excess rows and columns is pretty easy. (I could even make the table auto-shrink when you navigate back, like note really committing the changes to the size until you type something.)

Initially, I thought the "append row at bottom" and "append column to the right" actions would suffice for most people's needs. Having shortcuts to trigger these actions makes it super easy to grow a table, too, I found. But the current proposal may result in even less work on the user's side. There's no need to think about an action or search for a menu item with a proper label. The workload of simple pressing the right arrow key again to grow the table is so much smaller. It's too appealing to not at least try it out.

[tableflip]: http://tableflipapp.com/
[pruning]: /posts/2016/07/reswift-action-ui-events/
