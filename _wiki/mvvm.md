---
title: MVVM
---

"MVVM" stands for Model--View--View-Model. It's a [design pattern](/wiki/design_pattern/) which helps reduce the work of a view controller in Cocoa applications using AppKit or UIKit.

There are many different flavors. It makes a difference if you are developing an application with a [reactive](/wiki/reactive-programming/) framework in mind, where view models sometimes do most of the heavy lifting, or if you use the term to describe [data-transfer objects](/wiki/dto/).

No matter what, [MVVM is not an architecture.](/posts/2016/10/mvvms-place/) Don't let anybody tell you otherwise.

## Non-Reactive View Models

I recognize (see ["So I've finished my model, now what?"](/posts/2015/06/start-mvvm/)) two kinds of view models:

* **Active View Model:** changes to the view model automatically result in updates of the view. The view model _pushes_ changes to the view component or the view component _reacts_ to changes of the view model.
* **Passive View Model:** data-transfer objects; you have to update the view with a new view model to perform visible changes.

In both cases, the view model is an abstraction of whatever is going on deep inside your app's core tailor-made for the view it's used in.

For example, if your view contains labels, its view model should provide `String`-based accessors. And if the label contains a text you know that needs to be computed, the most straigth-forward solution is to expose the computation's result in the view model. From the view's perspective, it doesn't matter much if the view model assembles it on the fly and knows about the structure of the underlying data, or if it only contains stored properties that have to be assembled elsewhere:

```swift
// Core model
class PersonEntity {
    let age: UInt
    let firstName: String
    let lastName: String
}

// View-specific model
struct PersonViewModel {
    let age: String
    let fullName: String
}

extension PersonViewModel {
    init(fromPerson person: PersonEntity) {
        self.age = String(person.age)
        self.fullName = "\(person.lastName), \(person.firstName)"
    }
}
```
