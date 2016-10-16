---
title: Presenter and MVVM for UITableViews?
created_at: 2015-11-25 12:07:57 +0100
kind: worklog
tags: [ uitableview, mvvm, presenter ]
comments: on
---

When I wrote about [MVVM with a "control"](/posts/2015/10/view-model-control/) which really was a presenter, the view in question displayed some kind of data. Any sufficiently complex view would do, but the example focused on a view controller which consisted of a single composite view.

`UITableViewController`s are a different kind of beast, though.

Say you have three kinds of cells, each appears only once, and each has a different control (label or button), the resulting view will look much like a single composite view with 3 elements. But it's a table. And that makes things difficult, because you don't have convenient `@IBOutlet` references from the view controller.

The analogy is more like this: each cell is its own presentation control a.k.a. presenter. Subclassing in Swift is easy and it's cheap, so each cell type can be its own class with its own `@IBOutlet`s. 

But some of the outlets are buttons, as I said. The cell will receive the events first. How does it delegate actions up?

If the table is simple enough, you can use a protocol to handle all actions of the table. Then each cell calls the delegate method it needs and ignores the rest. That's not super sexy in terms of information hiding, but it works. Creating a protocol for each and every cell type could become worse.

For simple interactions, instead of protocols you can use closures as event handlers: `didPressFoo: (() -> Void)?`

Now each cell would have its own set of event handlers. That's great in terms of being specific, but now it couples the result of the table view data source more closely to the cells:

    #!swift
    let cell: GenericCustomCell = ...
    
    if let bananaCell = cell as? BananaCell {
        bananaCell.peelHandler = someDelegate.peelBanana()
    } else if let coconutCell = cell as? CoconutCell {
        coconutCell.crackOpenHandler = someDelegate.crackCoconut()
    }

That's not an option.

The best I could come up with is having both a protocol for all table interactions and event handlers. Then set up the cells through helper methods in extensions -- only that's not possible since the helper method has to be declared on the cells' shared type to call it without casting to concrete types:

    #!swift
    protocol GlobalEventHandler {
        func peelBanana()
        func crackCoconut()
    }
    
    protocol GenericCustomCell {
        // ...
        
        func populateHandlers(eventHandler: GlobalEventHandler)
    }
    
    class BananaCell {
        // ...
        
        weak var peelHandler: (() -> Void)?
        
        func populateHandlers(eventHandler: GlobalEventHandler) {
        
            peelHandler = eventHandler.peelBanana
        }
    }
    
    let cell: GenericCustomCell = ...
    cell.populateHandlers(someDelegate)

I need to use `GenericCustomCell` since I don't want to put knowledge about creating of concrete cells and their setup into one place.

Since with that version there's no real benefit over calling the delegate directly instead of closures, I am stuck at:

    #!swift
    protocol GlobalEventHandler {
        func peelBanana()
        func crackCoconut()
    }
    
    protocol GenericCustomCell {
        // ...
        
        var eventHandler: GlobalEventHandler { get set }
    }
        
    let cell: GenericCustomCell = ...
    cell.eventHandler = someDelegate

I have the feeling that I missed an opportunity to do it even better. 

If you got an idea, tell me in the comments!
