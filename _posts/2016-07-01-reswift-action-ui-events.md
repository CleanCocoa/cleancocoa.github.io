---
title: Separating ReSwift Actions from UI Events
created_at: 2016-07-01 16:58:18 +0200
kind: worklog
tags: [ reswift, tableflip, model, ddd ]
image: 201607011711_pruning.png
vgwort: http://vg01.met.vgwort.de/na/6b379fd672134e548793c6e71ee34ed7
comments: on
---

Today was the second time during the development of TableFlip that I started to implement a new feature in the wrong way: starting with an explicit event type that is triggered by pressing a button in the user interface. This is a 1:1 mapping of user intent to an event that performs changes in the model. Next time I'll start from another point of view instead to not rush too many minuscule changes until I hit a roadblock and hate myself. Here's what went wrong.

## Mapping to ReSwift Events

I want to implement _pruning_ in TableFlip. When you prune a table, empty columns and rows at the edges will be removed. This is handy because manually deleting multiple rows and columns is not much fun.

{% include figure.md src="/assets/blog/2016/201607011711_pruning.png" alt="pruned parts" caption="Rows and columns pruning will find and remove" %}

The algorithm can be simple. Start at the outside edges, work your way inward until the row or column isn't empty anymore. Take note of the row's (or column's) index, then remove all between there and the edge.

The `Table` model knows best how much it can remove inside its own boundaries. So the `prune()` command fits there nicely. Starting at a toolbar button, the flow of messages in [TableFlip's architecture][arch] is something like the following picture:

{% include figure.md src="/assets/blog/2016/201607011717_pruning-flow.png" alt="flow diagram" caption="Flow of events/messages from button click to state update" %}

So UI interactions are translated to ReSwift-compatible events which affect the model. My first hunch: model this as a simple `Prune` ReSwift action. There'll be no parameters since the `Table` model will figure out its own content boundaries.

Works well enough -- until you want to undo this with a simple inverted action.

[arch]: /posts/2016/06/tableflip-architecture/

## Action Inversions Break This

TableFlip deals with the undo stack in a rather simple way: for each action there's an inverted action. Remove column at index #5? Then undo-ing will insert a column after index #4. Very simple.

Removing rows and columns is simple. Inserting an empty column is simple, too. But you want the old contents back when you undo. So these inversions refer to a "past state" context. I lied, it's not the past state, really -- it's the current one. Each action's inversion is put onto the undo stack while the action is processed, so the context can simply return the current data which is kept in the inverted action for reference. 

In code, it looks like this:

    #!swift
    switch action {
    // ...
    case let .RemoveColumn(index): 
        let oldContents = context.column(index: index)
        return TableAction.InsertColumnBefore(index, contents: .Filled(oldContents))
    }

This works solely because the `.RemoveColumn` action has an associated index value which can be used for `.InsertColumnBefore` again.

But a pruning action doesn't _know_ the columns and rows which are going to be removed the way I modeled it.

And herein lies my big mistake: I made a 1:1 translation of user interface interaction to (ReSwift) events that change the model.

## Modeling Undoable Pruning Actions

When "pruning" is the user's intent, the action that is performed in the model can be called "cutout." It's inversion can be called "pad." 

The relation of "prune" and "cutout" is similar to "inverse selection" in a drawing application. I can use the existing pruning algorithm to determine useless areas in a table. Then I invert the affected area so I end up with two coordinates: where the content starts and where it ends.

Let's say the first coordinate is `topLeft = (column: 10, row: 4)`. It's trivial to determine the left padding for this action's inversion: `topLeft.column.predecessor().value` -- which will produce "9".

Performing a cutout on the table model will be implemented by calling `removeColumn` from indexes `1...9`. So the actual removal is closer to the existing mental model of "pruning" than it is to "cutting-out." That doesn't matter, though: this is the distinction between problem space ("cutout") and solution space ("remove until"). All is well.

{% include figure.md src="/assets/blog/2016/201607011758_pruning-flow-revised.png" alt="revised flow" caption="Flow of events after acknowledging two kinds of events (with numbers to un-confuse you about the order of messages sent)" %}

Inverting a "pad" action to "cutout" again is similarly simple.

So I end up with two kinds of events, for the second time as I said in the first paragraph:

1. UI events: "Prune"
2. ReSwift/Model events: "Cutout" and "Pad"

Some UI interactions can be mapped 1:1 to events in the model space just fine. Removing columns is one example. Appending a row at the bottom is even simpler. But this is not the direction I think I should design app changes.

The behavior-rich Domain Model of TableFlip is the app's premier point of change. I forgot to recognize this because "pruning" sounded so simple, and hey, I know how the app is made, so let's take a shortcut here and there and-- well, you know how this story ended.

When you invent a new way to interact with the app, meaningful change will affect the model. I was reminded to start with changes in the innermost core, the model, and work my way outward, no matter how simple the change sounds. Everything else is just too error-prone. 
