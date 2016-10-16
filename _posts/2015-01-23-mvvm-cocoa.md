---
title: Model–View–View Model (MVVM) Primer
created_at: 2015-01-23 10:57:20 +0100
kind: worklog
tags: [ mvvm, design-pattern, software-architecture, presenter ]
shared_image: macappdev-book.jpg
vgwort: http://vg08.met.vgwort.de/na/57cb1d6ce2354857a64d7d44e03711b3
comments: on
---

_This is an excerpt of [my book "Exploring Mac App Development Strategies"][book], on a sale right now!_

I learned that the "view model" in the architectural style of MVVM is a "model of the view" instead of a "model for the view". Let me explain the difference and the benefits of being able to chose between both.

There's your core _domain model_. Around it, you build an application with its user interface. The user interface is called _view_. So far, so similar to MVC.

Now an MVC controller will usually have access to the model data and the view, or the view will be bound to the model and update itself automatically on model changes. The latter is sometimes called _smart view pattern_. As we'll see, I don't like either of that. A basic Domain-Driven Design principle I adhere to is this: domain objects should not bleed into other layers of the application. At least not if they're mutable, that is, if they're Aggregates. You can argue that read-only entities would do no harm, but I yet have to come across something like this.

To get any data into the view, MVVM introduces the _view model_ or [_presentation model_][presmodel]. Application of this pattern vary, but they all have in common that the view model is a container for the data of the domain model. The domain model is thus shielded from the user interface. I came to think that the view model is "a model for the view", but people repeatedly say it's a "model of the view" instead. Depending on your interpretation, how you apply the pattern will vary of course.

* Dumb [Data-Transfer Objects][dto] are **"a model for the view"**. The view takes the data, works with it, and displays it somehow.
* Presenters, however, model what the view can do. They become **"a model of the view."** They take care of assembling data suited to the view's needs.

In the Ruby on Rails world, the [presenter][] has become a service object which holds on to the domain model and exposes its data in a way meaningful to the view. Say you've got a `Person` entity with `firstName` and `lastName`. A `PersonPresenter` will take care of assembling a `fullName`, to display a list of `Person`s, for example. You can go all nuts here: create a `PersonListPresenter` for displaying full names in a list, and a `PersonFormPresenter` for exposing both first and last name to edit them in a form, say. This way, the view can be as dumb as possible. That's a case of preparing "a model for the view." After all, you pass the presenter in, so it becomes the view's model.

I learned to think about it from a different perspective.

Adhering to dependency inversion, the concrete presenter is indeed injected into the view. Its interface is specified by means of the view component arrangement, though. If the view displays the full name of a person, the presenter's interface will codify this requirement as `fullName() -> String`. This way, **the view is responsible for specifying its data provider's capabilities** while it depends on some other service to provide an actual implementation. The concrete presenter is created outside the view layer.

Your client code doesn't need to know what the view does or which components it's made of. It only needs to understand the presenter's interface. The presenter in this case is a model _of_ the view: it doesn't care about the Domain Model. It only cares about the view's specification.

A few weeks ago, I [linked to](/posts/2014/12/mvvm-in-swift) a post by Srdan Rasic about MVVM in Swift. Adapted to the example here, it could look like this:

    #!swift
    class PersonViewController {
      var fullNameLabel: UILabel
   
      var viewModel: PersonViewViewModel {
        didSet {
          viewModel.fullName.bindAndFire {
            [unowned self] in
            self.fullNameLabel.text = $0
          }
        }
      }
      // ...
    }
    
Now when the view model's `fullName` property changes, the user interface will update. As a corollary, change the properties on this _model of the view_ to update the UI. A presenter hides away the complexity of the user interface. To the client code, the rest happens magically.

Essentially, that's what Cocoa Bindings promise. Only they tend to be used on objects of the Domain Model. That's not where they belong. Read [my book about clean coding for the Mac][book] to find out what to do instead!

<%= advertise :macappdevbook %>

[presmodel]: http://martinfowler.com/eaaDev/PresentationModel.html
[dto]: http://en.wikipedia.org/wiki/Data_transfer_object
[book]: https://leanpub.com/develop-mac-apps-clean-architecture-swift/
[presenter]: http://eewang.github.io/blog/2013/09/26/presenting-the-rails-presenter-pattern/
