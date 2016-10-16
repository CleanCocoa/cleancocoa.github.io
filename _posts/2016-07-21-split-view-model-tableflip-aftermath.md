---
title: "Splitting the View Models of TableFlip, a Short Aftermath"
created_at: 2016-07-21 18:42:39 +0200
kind: worklog
tags: [ 1df, refactoring, mvvm, presenter ]
image: 201607210932_tableflip.png
comments: on
---

I executed [my plan from earlier this week][1] to split TableFlip's monolithic view model into sub-view models. The process wasn't too complicated. The highlights:

* I found 3 quick fixes that I never got rid of, where a view controller reached 2+ levels deep into the main (and only) view model. 
* I reduced the `TableViewController` by 60 lines of code, moving 3 object dependencies to the new `SelectionViewController`.

The current hierarchy of controllers in the view layer is as follows:

* `TableWindowController`, which acts as a Facade, implementing `DisplaysDocument`, `DisplaysTable`, and `DisplaysSelection`.
    * `TabBarController: DisplaysDocument` shows tabs to switch between multiple tables in a document.
    * `TableViewController: DisplaysTable` takes care of updating the table view and reacting to some user interactions.
    * `SelectionViewController: DisplaysSelection` moves the selection highlighter.

{% include figure.md src="/assets/blog/2016/201607210932_tableflip.png" alt="screenshot of TableFlip" url="http://tableflipapp.com/" caption="Screenshot of TableFlip v0.6" %}

The window doesn't _look_ too complicated and there truly are not that many view components involved, but still it's quite some work to keep things well coordinated. The `TableViewController` file clocks in at about 400 lines of code although it mostly delegates stuff to other objects. There's still room for improvement, but I have to see patterns, first. I even extracted `NSTableViewaDataSource` and `NSTableViewDelegate` into different objects months ago. Today I doubt this was a good idea in the first place. We'll see.

Now that the single app state results in updates of 3 different view models, I can begin to plan performance optimizations: did the selection change? -- then don't redraw the table at all.

I love creating complex models and write unit tests for them. I really do. The view stuff, not so much. But now things really get interesting again as I try to improve the user experience and sometimes have to roll custom solutions instead of using the default AppKit stuff. The level of challenge increases. That's always nice.

[1]: /posts/2016/07/split-view-model-tableflip/
