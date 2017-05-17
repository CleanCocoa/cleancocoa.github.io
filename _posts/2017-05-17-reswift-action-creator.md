---
title: "Do You Need to Use Action Creators in ReSwift to Conditional Action Dispatching? (No)"
created_at: 2017-05-17 20:12:37 +0200
kind: worklog
tags: [ reswift ]
url: http://stackoverflow.com/questions/43831494/when-why-and-how-to-use-action-creators-in-redux/44032066#44032066
vgwort: http://vg01.met.vgwort.de/na/34658d66dae34bbb94eb39c741d4cc7e
comments: on
---

This is an answer to a StackOverflow question titled ["When, why and how to use Action Creators in redux?"](http://stackoverflow.com/questions/43831494/when-why-and-how-to-use-action-creators-in-redux/44032066#44032066) that is really targeted at ReSwift. The real question is phrased a bit differently, though:

> For example, I have a button, after user pressed it I want to start some process if I'm in state A. So, I have to write an action creator, which will check current state and then return correct action, or not action at all. Then dispatch this action from the same place.

Redux.js and ReSwift behave quite differently. [Redux recommendations](http://blog.isquaredsoftware.com/2016/10/idiomatic-redux-why-use-action-creators/) may not apply in all cases.


## Clearing up Redux.js/ReSwift confusions

### Action Creators in ReSwift

"Action creators" is a rather specialized term in Redux.js, but the type is defined like this in ReSwift as of v4:

```swift
public typealias ActionCreator = (_ state: State, _ store: Store) -> Action?
```

So contrary to general advice, you _do_ have access to the state at the point of dispatching the action. You don't need a Thunk implementation, although Thunks or Epics can help in more complex cases.

### "Action Creators help encapsulate action creation details"

A common benefit of Action Creators in Redux is their function as a factory. 

Redux.js actions are object literals. Redux actions, if you were to write them in Swift, would be more like `Dictionary`s. If you come from an Objective-C background, you know both the good and bad of dictionaries: they are flexible, but you can cause trouble with typos, and the compiler won't catch changes to values or keys if they are stringified. That's how Action Creators in Redux.js provide the convenience of factories (as in "Factory", the *Gang of Four* design pattern).

Here's one as an example:

```js
function addTodo(text) {
    return {
        type : "ADD_TODO",
        text
    }
}

// Somewhere else
dispatch(addTodo(text))
```


The `ReSwift.Action` type is usually implemented in custom value types (`struct`s), though. ReSwift does not suffer from that problems of Redux.js actions. That means by creating a custom action _type_ in ReSwift, the benefit of centralizing action creation in one function goes away. The initializer of your type does provide this already.

That paints a totally different picture.

And that's probably why `ReSwift.Store.ActionCreator` passes in the state: to provide any benefit at all. At the cost of a different kind of API than Redux.

## Application to the Question

Recall the question:

> For example, I have a button, after user pressed it I want to start some process if I'm in state A. So, I have to write an action creator, which will check current state and then return correct action, or not action at all. Then dispatch this action from the same place.

There are a couple of ways to achieve that.

If you have access to the store variable to call `dispatch`, you also have access to its current `state` property. You can ask the store for the state the app is in and act accordingly. Usually, you'd write store subscribers to get "push notifications" of store changes, but in cases like this you can also ask the store.

That means the following would be a totally valid implementation:

```swift
let currentState = store.state
let action: Action = {
    if currentState.someSubstate == "A" {
        return ActionWhenInStateA()
    } else {
        return ActionWhenNotInStateA()
    }
}
store.dispatch(action)
```

Since ReSwift `Store`s are not supposed to receive dispatch commands from different threads, you can rely on the state from line 1 to be the same in the last line, where you dispatch.

**TL;DR:** you do not need `ActionCreator` to achieve this. But you can if you like to write "east-oriented" code, leaning on callbacks instead of property queries:

```
store.dispatch { state, _ in
    if state.someSubstate == "A" {
        return ActionWhenInStateA()
    } else {
        return ActionWhenNotInStateA()
    }
}
```
