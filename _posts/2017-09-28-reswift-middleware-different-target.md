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
ApplicationMiddlewareRepository.instance
    .addBeforeReducers { (bananaAction: BananaAction) in 
        Swift.print("Banana action passing through: \(bananaAction)") }
```

As a bonus (and risk!), this approach allows you to change which middleware are active during runtime. Keep in mind that ReSwift is designed not to allow this, so proceed with caution.

The closure is of type `ApplicationMiddleware`. I found it beneficial to not have `ApplicationMiddleware` take any `ReSwift.Action` by default, because I usually conditionally check for one specific type, not multiple. So I lift the action type filtering to a generic constraint:

```swift
typealias ApplicationMiddleware<A : Action> = (A) -> Void
```

Now you cannot create an array from these function bodies anymore; also, though the `Action` subtype is now reified as a generic constraint, the actual cast from `Action` to `A` has to go somewhere. I put it into a `AnyApplicationMiddleware` box:

```swift
struct AnyApplicationMiddleware {
    let boxedMiddleware: (Action) -> Void

    init<A: Action>(_ base: @escaping ApplicationMiddleware<A>) {
        self.boxedMiddleware = { action in
            guard let castAction = action as? A else { return }
            base(castAction)
        }
    }
}
```

Now you can have an `[AnyApplicationMiddleware]` array and pass any `Action` through. Only matching actions will be passed to interested middleware.

Heads up: This closure-based approach can allow removing all registered `ApplicationMiddleware`, but not specific instances. For that, instead of a `typealias`'d closure I'd use a reference type and check for reference equality (`===`) to remove/deactivate them.

## The Full Implementation

```swift
public typealias ApplicationMiddleware<A : Action> = (A) -> Void

struct AnyApplicationMiddleware {

    let boxedMiddleware: (Action) -> Void

    init<A: Action>(_ base: @escaping ApplicationMiddleware<A>) {

        self.boxedMiddleware = { action in
            guard let castAction = action as? A else { return }
            base(castAction)
        }
    }

    func process(action: Action) {
        boxedMiddleware(action)
    }
}

public class ApplicationMiddlewareRepository {

    public static var instance = ApplicationMiddlewareRepository()
    private init() { }

    internal fileprivate(set) var middlewares: [AnyApplicationMiddleware] = []

    public func addBeforeReducers<A: Action>(middleware: @escaping ApplicationMiddleware<A>) {

        middlewares.append(AnyApplicationMiddleware(middleware))
    }

    internal func process(action: Action) {

        middlewares.forEach { $0.process(action: action) }
    }
}

let applicationMiddleware: Middleware<AppState> = { dispatch, getState in

    return { next in
        return { action in

            ApplicationMiddlewareRepository.instance
                .process(action: action)

            next(action)
        }
    }
}
```
