---
title: Refactoring a View Controller for Clean Information Flow
created_at: 2015-07-18 13:58:15 +0200
kind: worklog
tags: [ view-controller, refactoring ]
image: 201507111501_refactor.png
vgwort: http://vg08.met.vgwort.de/na/04b1af7ae8444c34ab781bf8a0c9a4e0
comments: on
---

There's a WWDC 2014 talk called ["Advanced iOS Application Architecture and Patterns"][mat]. In the first 30 minutes, you can learn a lot about designing information flow in your app.

Sticking to Andy Matuschak's example, a view controller is usually _the_ place to put all behavior. (Hint: this is a bad idea.)


* It has **model data**, which can be as simple as properties like `postText: String`.
* It has an outlet to a **text field** which should display the `postText`. 
* It also displays a **character count** using an outlet to a label.

If the text field's content is changed by the user, the character count needs to be updated. Since we're in the view controller anyway, we end up using the delegate method `textFieldDidChange(_:)` on iOS, for example. There, we pull the text field's contents and update the character count. We also set the model data to the new value.

Everything runs smooth as long as we don't have to deal with displaying existing text, or manipulating text without the user's interference, because then the delegate methods will not be called.

Discovering this bug, we can sketch the information flow according to Andy as follows:

{% include figure.md src="/assets/blog/2015/201507111507_original.png" alt="information flow" caption="Information flow: character count depends in part on the text field." %}

The seemingly convenient way to use the view controller instead let us hit the wall. We could work around the limitations of the delegate method, though: when we change the text field's content programmatically, we also have to issue an update of the character count.

Now, at last, your alarm bells should ring. If we have to ensure we change two values in lockstep whenever we change one of them, we introduce coupling. Calling two methods in succession doesn't show the coupling, though. A cautious reader of the code may notice, or we as author could leave comments behind, but that's not how you design things that belong together. 

The authority of what the content looks like should be the model, both for the text field's contents and for the character count.

{% include figure.md src="/assets/blog/2015/201507111501_refactor.png" alt="refactored information flow" caption="Make the model the sole authority and propagate change events to interested parties (i.e. the view controller)." %}

To make this explicit, the following can be part of a first step to clean the code up:

* Create a higher level method on the view controller. Let's call it `contentDidChange(_:)` to adhere to delegate conventions.
* Create a property observer or a similar mechanism on the model to call this method upon changes. A `didSet` property observer in Swift on the `postText` property can do just that. KVO in Objective-C may help, too. If your model is a custom object, you need to [bind to its properties](/posts/2014/12/mvvm-in-swift/) instead of the model itself.

Now there's one central place where the view content is updated. There, you put both setting the text field's contents and updating the character count.

I used to shy away from creating protocols to define delegate behavior. Nowadays, I prefer to write one protocol too many at first and remove it later than introduce cross-dependencies which it's hard to reason about.

[mat]: https://developer.apple.com/videos/wwdc/2014/
