---
title: How to Make ReSwift Actions Undoable with NSUndoManager
created_at: 2017-03-01 12:50:26 +0100
kind: worklog
tags: [ reswift, undo ]
vgwort: http://vg01.met.vgwort.de/na/daf73025ced64ed6813987e4dae8f39f
comments: on
---

I wrote about [using `NSUndoManager` + ReSwift with a Middleware](/posts/2016/08/reswift-undo-middleware/). You could think of the resulting `UndoMiddleware` as some kind of observer of events that pass the `ReSwift.Store` and which puts the _opposites_ of incoming actions on the undo stack. Why a Middleware? Because making an action undoable is a side effect.

The Middleware wasn't exactly straightforward. It took a bit of type wrapping and boilerplate code. A "context provider" made parts of the current app state and model available to compute the opposite action. Without this context, the Middleware couldn't know what was expected.

Today I came up with something shorter: an _action creator._ It works well because the actions in my current project are simple and easily reversed.

Instead of dispatching an action like this:

    #!swift
    store.dispatch(CreateFoo(id: 1))

... you create the opposite of the action where you already do have knowledge about the context:
    
    #!swift
    let creation     = CreateFoo(id: 1, content: "...")
    let undoCreation = DeleteFoo(id : 1)
    store.dispatch(undoable(creation, opposite: undoCreation))

Of course it takes more effort to compute it the other way around because you have to fetch the current content before you delete (that's what the context provider of my Middleware did, too):
    
    #!swift
    let id = // ...
    let oldContent = foo(withId: id).content
    let deletion =     DeleteFoo(id: id)
    let undoDeletion = CreateFoo(id: id, content: content)
    store.dispatch(undoable(deletion, opposite: undoDeletion))

Chances are you have a lot of the pieces of the puzzle to assemble the "undo" action in the service object that dispatches the original action. In contrast, the Middleware knew nothing and thus had to depend on another source of information for everything.

The `undoable` action creator is very simple, after all:

    #!swift
    public func undoable(_ action: Action, opposite: Action?)
        -> (AppState, DefaultStore)
        -> Action?
    {
        return { (appState: AppState, store: DefaultStore) -> Action? in
        
            if let undoAction = opposite,
                
                // Replace this with something useful in your app:
                let undoManager = getUndoManagerFromSomewhere() {
                
                undoManager.registerUndo(withTarget: store) { store in
                    store.dispatch(undoable(undoAction, opposite: action))
                }
            }
        
            return action
        }
    }

This assumes symmetry of actions: inside the `registerUndo` block, `undoable` is called again, only the other way around, to setup what "redo" should be like. If your original action is not what you want to use for redoing, then you need to put that in somehow.

This is the case in my app where `DeleteFoo` creates a [pending file change](/posts/2017/02/reswift-enqueue-file-changes/) that ends with dispatching `DeleteFooCompleted`. The opposite of `DeleteFooCompleted` is `CreateFoo` -- but the opposite of `CreateFoo` is `DeleteFoo`, not `DeleteFooCompleted`, so I put a mapping in between:

    #!swift
    public func redoable(basedOn action: Action) -> Action {
        switch action {
        case let action as DeleteFooCompleted:
            return DeletingFoo(id: action.id)

        default:
            return action
        }
    }

    public func undoable(_ action: Action, opposite: Action?)
        -> (AppState, DefaultStore)
        -> Action?
    {
        return { (appState: AppState, store: DefaultStore) -> Action? in
        
            if let undoAction = opposite,
                let undoManager = getUndoManagerFromSomewhere() {

                // Original trigger might be a 'completion' event that has to be
                // "redone" using a 'starting' event.
                let redoAction = redoable(basedOn: action)

                undoRegistrar.registerUndo(withTarget: store) { store in
                    store.dispatch(undoable(undoAction, opposite: redoAction))
                }
            }
        
            return action
        }
    }

And that's it!

The biggest problem is to obtain a reference to the main window's `UndoManager` instance. You can inject it once but not replace it later, which I would have needed, so I end up with a getter here, which I don't like a lot; you might be fine with providing a custom `UndoManager` instance during setup, though! In `NSDocument`-based apps, where each document has its own window with its own undo manager and probably its own store instance, it'll be pretty easy.
