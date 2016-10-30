---
title: "Resolving NSTreeController's \"Ambiguous use of 'children'\" in Swift 3"
created_at: 2016-10-27 16:34:56 +0200
kind: worklog
tags: [ swift ]
comments: on
---

I'm converting code for [my first book](https://leanpub.com/develop-mac-apps-clean-architecture-swift) to Swift 3. It uses Cocoa Bindings a lot, including `NSTreeController`.

Now the compiler changed; and one of the issues I faced was working with the `NSTreeController.arrangedObjects`. The compiler assumed a wrong type of the `children` property (see [the docs](https://developer.apple.com/reference/appkit/nstreecontroller/1527465-arrangedobjects#)) and reported "Ambiguous use of 'children'".

Thanks to [StackOverflow](http://stackoverflow.com/questions/39660939/ambiguous-use-of-children-when-trying-to-use-nstreecontroller-arrangedobject), I found out that specific casting is supposed to help. The disambiguation didn't work for me though.

My resulting code looks like this:

    #!swift
    let itemsController: NSTreeController = ...
    let treeNodes: [NSTreeNode] = itemsController.arrangedObjects as AnyObject).children!!

To make this work, I had to move my custom tree node type _to a different `.swift` source file._ That type contains a `children` property to comply to `NSTreeNode`'s requirements. Thus the compiler assumed the unknown `arrangedObjects as AnyObject` cast really meant my own custom type. (That's my explanation at least.)

For reference, this is the type I'm talking about:

    #!swift
    @objc public protocol TreeNode {
        var title: String { get set }
        var count: UInt { get set }
        var children: [TreeNode] { get }
        var isLeaf: Bool { get }
        weak var eventHandler: HandlesItemListChanges? { get }
        func resetTitleToDefaultTitle()
    }

So if your Swift 3 conversion doesn't work as expected and your `NSTreeController`/`NSArrayController`/`NSPageController` is causing trouble ...

1. move any type with a `children` property to a different file,
2. ensure you are specifying that you expect the children to be of type `[NSTreeNode]` -- either through a cast (`.children as [NSTreeNode]?`), through specifying a helper function's return type, or though specifying the expected variable type.

Help your compiler infer what's going on in this still-very-weird casting situation.
