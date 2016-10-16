---
title: Putting ReSwift Actions on the Undo Stack Using Middleware
created_at: 2016-08-17 11:05:58 +0200
kind: worklog
tags: [ reswift, undo, middleware ]
vgwort: http://vg01.met.vgwort.de/na/666920f1790d40ea84fb7bb6e2b6fbea
comments: on
---

It took me a while to grok the use of "Middleware" in ReSwift. [ReSwift's own tests][1] illustrate what you can do with it: you can filter or modify actions before they are passed to the Reducers, for example.

I use this in TableFlip to modify the undo stack now. Every action that's undo-able is registered with the app's `NSUndoManager`.

[1]: https://github.com/ReSwift/ReSwift/blob/master/ReSwiftTests/StoreMiddlewareTests.swift

Take this enum of actions, for example:

    #!swift
    struct AppState: ReSwift.StateType {
        var content: String = ""
    }
    
    enum DocumentAction: ReSwift.Action {
        case replaceContent(String, isInitialContent: Bool)
    }

When a document is loaded, `.replaceContent` is dispatched with `isInitialContent` set to `true`. This flag is useful to not make the initial display of data undo-able. I'll show you how I use that later. Keep in mind that when I initially created an action similar to this one, the `isInitialContent` flag wasn't there. I discovered the need for it only later in the process.

"Doing" actions in the context of ReSwift means you invoke a Store's `dispatch` method with a supported action. Like this:

    #!swift
    let action = DocumentAction.replaceContent("new content", isInitialContent: false)
    store.dispatch(action)

This is supposed to change the content of the document. Maybe a text field displays "new content" now; maybe it also changes the contents of a file or fires off a network request. That doesn't matter right now. All that matters is that this action is triggered, and that I want to undo it.

Enter `NSUndoManager`. Registering an action with the `NSUndoManager` is as simple as calling `registerUndoWithTarget(_:, selector:, object:)`. But you'll have to do this for the redo counterpart as well. Traditionally, people put it into the same function:

    #!swift
    class AnObject: NSObject {
        let undoManager = ... // maybe a property of the object
        func doSomething() {
            if undoManager.undoing {
                undoManager.registerUndoWithTarget(self, selector: #selector(doSomething), object: nil)
            }
            undoManager.registerUndoWithTarget(self, selector: #selector(doSomething), object: nil)
        }
    }

This easily gets out of hand. 

Let's segue to registering undo actions in a bearable fashion before we continue to deal with the undo middleware.

## Controlling What's on the Undo Stack

`NSUndoManager` doesn't accept the `DocumentAction` itself. Its API utilizes ye olde target--action to manipulate the undo stack. To transform my internal events to things that can be undone, I figured that blocks would be a good abstraction:

    #!swift
    class UndoAction: NSObject {

        typealias Block = () -> Void

        let undoBlock: Block
        let undoName: String?
        let redoBlock: (Block)?

        init(undoBlock: Block, undoName: String? = nil, redoBlock: (Block)? = nil) {

            self.undoBlock = undoBlock
            self.undoName = undoName
            self.redoBlock = redoBlock
        }

        var inverted: UndoAction? {

            guard let redoBlock = self.redoBlock else { return nil }

            // No need to pass a `undoName` as it will be associated with the current action
            // by the NSUndoManager.
            return UndoAction(undoBlock: redoBlock, redoBlock: undoBlock)
        }
    }

The `inverted` property switches the undo and redo block. That's it.

To put an `UndoAction` on the stack, I don't use the `undoBlock` directly because that doesn't work. Instead, I register a callback which then invokes the `undoBlock`:
    
    #!swift
    extension NSUndoManager {

        func registerUndo(action action: UndoAction) {

            registerUndoWithTarget(self, selector: #selector(performUndo(_:)), object: action)
        }

        @objc private func performUndo(action: UndoAction) {

            action.undoBlock()

            if let inverted = action.inverted {
                // When you `registerUndoWithTarget` while undoing, the result is 
                // actually put on the "Redo" side. ¯\_(ツ)_/¯
                registerUndo(action: inverted)
            }
        }
    }

That'll work. But I want to set the caption of the undo-able action in the app's Edit menu, too, without doing much on the client side, so I utilize the amazing "double dispatch" technique -- which is fancy talk for take a dependency as method parameter and send it a message with your property:

    #!swift
    extension UndoAction {
        func register(undoManager undoManager: NSUndoManager) {
            undoManager.registerUndo(action: self)

            // Set the action name in the menu, but don't overwrite
            // the inferred name for the redo counterpart.
            if !undoManager.undoing,
                let undoName = self.undoName {

                undoManager.setActionName(undoName)
            }
        }
    }

With that stuff in place, registering an action is as simple as this:

    #!swift
    let undoManager = ...
    let action = UndoAction(undoBlock: { print("undone!") })
    action.register(undoManager: undoManager)

Now back to ReSwift.

## Inverting User-Triggered Events

For simplicity's sake, I only put a `DocumentAction.replaceContent` in this example. This particular action can be used in three cases:

1. To display initial contents (not undoable),
2. to show different contents in the same document window (undoable), and
3. to undo the change and show the old contents again (redoable).

The first two cases are easy. We just have to dispatch them to the Store so the event is passed to the Reducers which in turn modify the app state. Dispatching is a one-liner for each case:

    #!swift
    // Show initial content
    store.dispatch(DocumentAction.replaceContent("Initial content", isInitialContent: true))
    
    // Replace content because of a server callback, user request, or whatever
    store.dispatch(DocumentAction.replaceContent("Replacement", isInitialContent: false))

Now the undo middleware should register the _inverted_ action to the undo stack. Creating an inversion is still missing. For 100% content replacements, this is easy: the inversion is a `.replaceContent` action with the content _before_ changes have applied. 

An exemplary story goes like this:

    #!swift
    // State = ""
    store.dispatch(DocumentAction.replaceContent("Initial content", isInitialContent: true))
    // → State = "Initial content"

    store.dispatch(DocumentAction.replaceContent("Replacement", isInitialContent: false))
    // → State = "Replacement"

    undoLastActionSomehow()
    // → State = "Initial content"

Here, `undoLastActionSomehow()` essentially equals sending `.replaceContent("Initial content", isInitialContent: false)` from the app state's perspective. _But_ the undo stack won't work if all we do is push new messages onto it. That's what the `undone` flag is good for: so that the inverted action does not end up on the undo stack, but is ignored.

Not putting the inverted `DocumentAction` on the undo stack again is important. `NSUndoManager` puts the inversion on the redo stack for you in `performUndo` (see code above).

Figuring out how to compute the inverted case is pretty easy in our case. It's the action's own responsibility to know how to do that, given a context which provides the data it needs.

    #!swift
    extension DocumentAction {
        func inverse(context state: AppState) -> DocumentAction? {

            switch self {
            case let .Replace(_, isInitial: isInitial):
                if isInitial { return nil }
                return DocumentAction.Replace(state.content, isInitial: isInitial)
            }
        }
    }

Of course TableFlip has a lot of actions that need more context than this in order to not replace 100% of the document with every change. 

Distinguishing between a regular action and an action that was triggered from the undo stack is important, so that when a user triggers "Undo Replace Content", for example, the same action isn't pushed to the undo stack again.

    #!swift
    /// Wrapper around `DocumentAction` to flag an action as already
    /// on the Undo-stack.
    private struct Undone: Action {
        let action: DocumentAction

        init(_ action: DocumentAction) {
            self.action = action
        }
    }

`Undone` is not itself the reversal of the action. (We created that above already.) It's just a simple marker. So there's not much logic involved to make this come together:

    #!swift
    private extension DocumentAction {

        var undone: Undone {
            return Undone(self)
        }

        var isUndoable: Bool {

            switch self {
            case let .replaceContent(_, isInitialContent: isInitial): return !isInitial
            default: return true
            }
        }
    }

I can now mark a copy of any `DocumentAction` as "on the undo stack" and determine if it _should_ be undoable with the `isUndoable` property.

This is how everything comes together:

    #!swift
    extension UndoAction {

        convenience init?(documentAction: DocumentAction, 
            context: AppState, 
            dispatch: ReSwift.DispatchFunction) {

            guard let inverseAction = documentAction.inverse(context: context)
                else { return nil }

            self.init(undoBlock: { dispatch(inverseAction.undone) },
                      redoBlock: { dispatch(documentAction.undone) })
        }
    }

With that in place, the (sole) `DocumentAction` can compute its inverse; the inversion is used to register undo actions with `NSUndoManager`. When the "Undo" menu item is clicked, the non-inverted original action will be put on the redo stack.

## The Undo Middleware

ReSwift's `Middleware` type is a bit ... confusing at first:

    #!swift
    public typealias Middleware = (ReSwift.DispatchFunction?, ReSwift.GetState) 
        -> ReSwift.DispatchFunction 
        -> ReSwift.DispatchFunction
    
    public typealias DispatchFunction = (Action) -> Any
    public typealias GetState = () -> StateType?

Returning a `DispatchFunction` from another `DispatchFunction` looked bogus to me at first: why return a closure that takes an `Action` and have it return another closure that takes an `Action`? -- I didn't realize this is a totally wrong interpretation until I read [the unit tests I initially mentioned.][1] 

A Middleware ends up looking like this, with lots of explicit type annotations for your orientation:

    #!swift
    let someMiddleware: Middleware = { (dispatch: ReSwift.DispatchFunction, getState: ReSwift.GetState) in
        return { (next: ReSwift.DispatchFunction) in
            return { (action: ReSwift.Action) in
                
                // Pass through without doing anything
                return next(action)
            }
        }
    }

You see that the innermost block is the actual transformation of type `DispatchFunction`, taking an action and returning a transformed state, for example. 

The block in the middle is not itself a `DispatchFunction` but it _takes_ one and it _returns_ one. The one it takes as parameter is the next middleware in the chain (or, in the cast of the last middleware, the invocation of the main Reducer), hence the name `next`. The one it returns is your transformation.

With this structure in mind, the undo middleware I use looks similar to this:

    #!swift
    let undoManager: NSUndoManager = ...
    let undoMiddleware: Middleware = { dispatch, getState in
        return { next in
            return { action in

                // Pass already undone actions through
                if let undoneAction = action as? Undone {
                    return next(undoneAction.action)
                }

                if let action = action as? DocumentAction 
                    where action.isUndoable,
                    
                    let state = getState() as? AppState,
                    let dispatch = dispatch,
                    
                    let undo = UndoAction(documentAction: action, context: state, dispatch: dispatch) {

                    undo.register(undoManager: undoManager)
                }

                return next(action)
            }
        }
    }

Here, we only try to register actions we control and ignore the `ReSwift.Init` action, for example, by passing it through.

---

This was quite some work to write a ReSwift Middleware which registers events with the `NSUndoManager`. A lot of the work was targeted at creating `UndoAction`, though, which can be used independently of ReSwift in your Cocoa projects to encapsulate undoable actions. The rest was tricky only because reversing content changes is a non-trivial task. With the example I created, it was very easy to implement `DocumentAction.inverse(context:)`.

The one thing that's on my mind now is this: doesn't the `NSUndoManager` own a piece of application state now? After all, it's the only object that has knowledge about both the undo and redo stack.

There's a proof-of-concept [ReSwift Recorder](https://github.com/ReSwift/ReSwift-Recorder) library which records events so you can time-travel from within the app. Time-traveling achieves the same effect as undo/redo. Ditching the Cocoa framework's well-established `NSUndoManager` for a proof-of-concept time travel library isn't worth it for me, although I'd have preferred the conceptual purity of recording past events purely using ReSwift components.
