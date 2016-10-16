---
title: Library for Providing Static UITableView Contents in Swift
created_at: 2015-07-28 15:50:35 +0200
kind: worklog
tags: [ functional-programming, swift, uitableview ]
comments: on
---

I discovered a Swift library called [Static][st] the other day. It helps set up and fill `UITableView`s with static content -- that is, it doesn't permit deletion or rearrangement of new rows into the table.

I think the following example to configure sections with rows reads really nice:

    Section(header: "Money", rows: [
        Row(text: "Balance", 
            detailText: "$12.00", 
            accessory: .DisclosureIndicator, 
            selection: {
                
            // Show statement
        }),
        Row(text: "Transfer to Bank…", 
            cellClass: ButtonCell.self, 
            selection: {
                
            // Show transfer to bank modal
        })
    ], footer: "Transfers usually arrive within 1-3 business days.")

This will work with data you fetch from Core Data or a server, too, of course. As long as the contents of the table don't need to change once it's set up, [Static][st] is a good choice!

When you're into creating view controllers programmatically for static content, here's a gentle reminder to have a look at Chris Eidhof's [functional view controllers on GitHub](https://github.com/chriseidhof/github-issues). Have a look at [his 30 minute talk at AltConf](https://realm.io/news/altconf-chris-eidhof-functional-programming-in-swift/) where he walks you through the process of setting up a sequence of view controllers and navigating through it.

I think Static is a great data source replacement and fits Chris' approach really well.

[st]: https://github.com/venmo/Static
