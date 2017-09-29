---
title: Inject ReSwift Middlewares from Different Project Targets During Runtime
created_at: 2017-09-28 10:45:13 +0200
kind: worklog
tags: [ reswift, middleware ]
comments: on
---

Say you extract the ReSwift-based state module with all your app's state types, reducers, and middleware into its own framework target. Until you add middlewares to the mix, it's easy to keep this coherent.

Injecting side-effects through middleware is not easy to do, though, as long as you provide the `Store` instance from your state module. The store takes an array of `Middleware` upon initialization. They cannot be changed later. The state module would then depend on your app module with all its network request services and whatnot. That's the wrong way around, also resulting in a circular dependency. Instead, you have two options to add side-effects that are inherently part of your app's module, not of the state module:

1. Create the store inside the app module, not in the state module;
2. Add a general-purpose side-effect middleware that exposes mutation functions to the outside world.

The latter can look like this:

```swift
SideEffectRegistry.instance
    .addBeforeReducers { (bananaAction: BananaAction) in 
        Swift.print("Banana action passing through: \(bananaAction)") }
```

As a bonus (and risk!), this approach allows you to change which middleware are active during runtime. Keep in mind that ReSwift is designed not to allow this, so proceed with caution.

The closure is of type `SideEffect`. I found it beneficial to not have `SideEffect` take just any `ReSwift.Action` by default, because I usually conditionally check for one specific type, not multiple. So I lift the action type filtering to a generic constraint:

```swift
public typealias SideEffect<A : Action> = (_ action: A, _ dispatch: DispatchFunction) -> Void
```

Now you cannot create an array from these function bodies anymore; also, though the `Action` subtype is now reified as a generic constraint, the actual cast from `Action` to `A` has to go somewhere. I put it into a `AnySideEffect` box:

```swift
struct AnySideEffect {
    let boxedSideEffect: (Action, DispatchFunction) -> Void

    init<A: Action>(_ base: @escaping SideEffect<A>) {
        self.boxedSideEffect = { action, dispatch in
            guard let castAction = action as? A else { return }
            base(castAction, dispatch)
        }
    }
}
```

Now you can have an `[AnySideEffect]` array and pass any `Action` through. Only matching actions will be passed to interested middleware.

Heads up: This closure-based approach can allow removing all registered `SideEffect`, but not specific instances. For that, instead of a `typealias`'d closure I'd use a reference type and check for reference equality (`===`) to remove/deactivate them.

**Update 2017-09-29**: I renamed the types a bit, from `ApplicationMiddleware` to `SideEffect`, etc.

## The Full Implementation

```swift
/// - parameter action: The expected action to perform a side effect on.
/// - parameter dispatch: Default dispatch function into the current store.
public typealias SideEffect<A : Action> = (_ action: A, _ dispatch: DispatchFunction) -> Void

struct AnySideEffect {
    let boxedSideEffect: (Action, DispatchFunction) -> Void

    init<A: Action>(_ base: @escaping SideEffect<A>) {
        self.boxedSideEffect = { action, dispatch in
            guard let castAction = action as? A else { return }
            base(castAction, dispatch)
        }
    }

    func process(action: Action, dispatch: DispatchFunction) {
        boxedSideEffect(action, dispatch)
    }
}

public class SideEffectRegistry {
    public static var instance = SideEffectRegistry()
    private init() { }

    internal fileprivate(set) var sideEffects: [AnySideEffect] = []

    public func addBeforeReducers<A: Action>(sideEffect: @escaping SideEffect<A>) {
        sideEffects.append(AnySideEffect(sideEffect))
    }

    internal func process(action: Action, dispatch: DispatchFunction) {
        sideEffects.forEach { $0.process(action: action, dispatch: dispatch) }
    }
}

let sideEffectsMiddleware: Middleware<AppState> = { dispatch, getState in
    return { next in
        return { action in
            SideEffectRegistry.instance
                .process(action: action, dispatch: dispatch)

            next(action)
        }
    }
}
```

