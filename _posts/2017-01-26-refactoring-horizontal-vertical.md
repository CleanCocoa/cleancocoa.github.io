---
title: "Refactoring – Extract Objects Horizontally or Vertically"
created_at: 2017-01-26 15:34:10 +0100
kind: worklog
tags: [ refactoring, presenter, view-model ]
vgwort: http://vg01.met.vgwort.de/na/ece1ea4e86764f85bcc8f103856f344f
comments: on
---


Let's say you have a `Presenter` that creates a `ViewModel` for its `View` from incoming data and then presents it in said view. 

In this case, the `Presenter` also is a `ReSwift.StoreSubscriber` for good measure, but it could receive the data by any means, really.

{% include figure.md src="/assets/blog/2017/1-original.png" alt="sketch of original setup" caption="Originally, the Presenter is a store subscriber that creates and presents a View Model" %}

Transformation from one model type to another is very simple at first. So you create a method to do the job:

```swift
class Presenter: ReSwift.StoreSubscriber {
    let view: View // ...
    
    func newState(_ state: AppState) {
        let viewModel = self.viewModel(from: state)
        view.update(viewModel)
    }
    
    fileprivate func viewModel(from state: AppState) -> ViewModel {
        return ViewModel(label: String(state.count))
    }
}
```

But since you're a well-behaved Swift developer and love extensions, you end up with another implementation that does the job equally well but closer adheres to your team's conventions

```swift
class Presenter: ReSwift.StoreSubscriber {
    let view: View // ...
    
    func newState(_ state: AppState) {
        let viewModel = ViewModel(from: state)
        view.update(viewModel)
    }
}
    
fileprivate extension ViewModel {
    init(from state: AppState) {
        self.label = String(state.count)
    }
}
```

Since both are private, it doesn't matter to the outside world. But the initializer is just as much a factory doing the job of adapting `AppState` to `ViewModel` as `viewModel(from:)`. If you (or your team) find these kinds of extensions confusing, then by all means: don't use them. Pick what suits your taste.

Now the `ViewModel` gets more complex and you get a bit more uncomfortable with the actual adaptation logic:

```swift
fileprivate extension ViewModel {
    init(from state: AppState) {
        if state.countIsVisible {
            self.label = String(state.count)
            self.actionText = "Increment!"
        } else {
            self.label = "Blank"
            self.actionText = "Start Counting"
        }
    }
}
```

Remember that this extension is still a factual private implementation detail of the `Presenter`, not of `ViewModel`, since nobody but the `Presenter` from the same file will ever know about it.

Your `Presenter` is doing a very complicated job now. Questions arise, like "Is this well factored?" -- There are numerous ways out of this situation if you aren't satisfied and want to refactor your code.

## Refactor Vertically

In this variant, you choose to tear the responsibilities apart at their functions. A "presenter" should do presenting, not complex mapping; so you introduce a `Mapper` that does just that. 

{% include figure.md src="/assets/blog/2017/2-vertical.png" alt="sketch of vertical refactoring" caption="View Model creation got so complex that a factory, aptly named 'Mapper', became necessary" %}

You could dub this the "Kingdom of Nouns"-approach or the Java-approach. That doesn't mean it's not a valid technique. 

Why "vertical"? When you scale up a server stack, you can scale it up vertically by adding more RAM or another disk, say. The amount of machines stays the same.

If you look at the sketch above again, you see that there's still only 1 flow of data, but it now carries to one more object.

## Refactor Horizontally

While "vertical" scaling of servers means make existing machines more powerful, "horizontal" scaling means you add another computer to the stack.

Tearing the responsibility apart horizontally then equals creating a separate stack starting at the presenter. 

{% include figure.md src="/assets/blog/2017/3-horizontal.png" alt="sketch of horizontal refactoring" caption="Tearing the view models and presenters apart instead." %}

In this approach, you have 2 and not 1 flow of data affecting what's on screen. 

The corresponding stacks may look something like this:

```swift
class LabelPresenter: ReSwift.StoreSubscriber {
    let labelView: LabelView // ...
    
    func newState(_ state: AppState) {
        let viewModel = LabelViewModel(from: state)
        labelView.update(labelViewModel: viewModel)
    }
}

fileprivate extension LabelViewModel {
    init(from state: AppState) {
        if state.countIsVisible {
            self.label = String(state.count)
        } else {
            self.label = "Blank"
        }
    }
}
```

... and one for the other:

```swift
class ActionPresenter: ReSwift.StoreSubscriber {
    let actionView: ActionView // ...
    
    func newState(_ state: AppState) {
        let viewModel = ActionViewModel(from: state)
        actionView.update(actionViewModel: viewModel)
    }
}

fileprivate extension ActionViewModel {
    init(from state: AppState) {
        if state.countIsVisible {
            self.actionText = "Increment!"
        } else {
            self.actionText = "Start Counting"
        }
    }
}
```

If you pay close attention, you will notice that I didn't just branch off into 2 presenters and 2 view models, but also into 2 view types. These would be protocols you implement in a single view component like a `UIViewController` which in turn delegates stuff down to the actual views, or you tie the presenters to these sub-views directly. That absolutely depends on the factoring of your user interface layer -- the presenters don't care as long as the protocol requirements are met.

## So in Which Direction Should I Refactor?

As always, this depends. 

* Vertical refactoring introduces new concepts into your code base. Readers now have to understand what a "mapper" does and keep more of these components in their heads when they try to make sense of the code. 
* Horizontal refactoring branches off early and forces readers to find all presenters that play a role in displaying a state update -- when you use ReSwift, that means both presenters react to the very _same_ state update, not just similar-looking ones.

Discuss this with your team and/or yourself. Which path do you find easier to follow? What will cause more confusion in a month or two?

I like to separate the interface early on and have multiple presenters; but if some of these present complex data, something like a `Mapper` can make sense. So this isn't necessarily an either-or question. Rather, it's a decision how you want to make the _next_ step today.

If your `Mapper` is a pure function transforming the state into a view model, then it doesn't really matter from a unit testing perspective since you'll have to exercise the mapping in presenter tests anyway. But even these implementation details matter when you read the production code.
