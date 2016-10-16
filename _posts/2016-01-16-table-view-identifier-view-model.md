---
title: How to Couple UITableView Cell Identifiers and View Models
created_at: 2016-01-16 13:36:48 +0100
kind: worklog
tags: [ mvvm, uitableview, view-model, oop ]
vgwort: http://vg01.met.vgwort.de/na/2d3eeb96e6e44e0f80cd7873837448cd
comments: on
---


Rui Peres [proposes](http://codeplease.io/2016/01/10/cells-and-viewmodels/) to make `UITableViewCell` view models the topmost model data. Traditionally, Cocoa developers stored _something_ in arrays that corresponded to the `indexPath` of a cell. In principle, this qualifies as "model" data already, but it's not yet a view model. In practice, it can even be something different than a view model entirely -- and make your view controllers slimmer!

First, let's have a look at Rui's code after introducing the view model, leaving aside the implementation details:

    #!swift
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        
        let viewModel = viewModels[indexPath.row]
        let identifier = viewModel.identifier
        let cell = tableView.dequeueReusableCellWithIdentifier(identifier, forIndexPath: indexPath) as! ChannelCell
        cell.viewModel = viewModel
        
        return cell
    }

You see that the `viewModel` knows about its cell's `identifier`.

I find that kind of troubling because now the view model has a little extra responsibility that's tied to a `UITableViewDataSource`: recognizing cell identifiers. The view model's sole responsibility should be to contain data that a cell displays. It shouldn't care much about the cell.

As a first step towards taking this responsibility away from the view model, I'd employ tuples:

    #!swift
    // Inside the view controller:
    typealias CellData = (viewModel: ViewModel, cellIdentifier: String)
    var viewModels: [CellData]

Now that there exists something dedicated to keeping identifiers and view data together, does this open up new design decisions? 

The next step would be to create a value type if I needed behavior on that. For example you could move the cell reconstitution and configuration out of the view controller and into the struct. The tuple with named parts already behaves like a simple struct when retrieving values. But if you need behavior, go for the struct:

    #!swift
    // Inside the view controller:
    struct CellSetup {
        
        let viewModel: ViewModel
        let cellIdentifier: String
        
        func dequeueCell(tableView: UITableView, 
            forIndexPath indexPath: NSIndexPath) -> UITableViewCell {
             
            let cell = tableView.dequeueReusableCellWithIdentifier(cellIdentifier, 
                 forIndexPath: indexPath)
            cell.configure(viewModel)
            return cell
        }
    }
    
    var cellSetups: [CellSetup]
    
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        
        let cellSetup = cellSetups[indexPath.row]
        return cellSetup.dequeueCell(tableView, forIndexPath: indexPath)
    }

I have changed the example to get rid of the `ChannelCell` cast -- because I think `CellSetup` better be made generic so that certain view model and cell pairs can be coupled and automatically be passed to each other in `CellSetup.dequeueCell`. I leave this as an exercise to the reader for now.
