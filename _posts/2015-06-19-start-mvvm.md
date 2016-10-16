---
title: "So I've finished my model, now what?"
created_at: 2015-06-19 14:33:10 +0200
kind: worklog
tags: [ mvvm, layered-architecture, viper, model ]
image: 201506191432_mvvm-arch.png
vgwort: http://vg08.met.vgwort.de/na/2cb7c31bd4e04c82a555d23da417753e
comments: on
---

After working on the model of your app for a while,  it can be hard to change your ways of thinking and see what should be coming next. How does an app come out of all this? Where does the user interface plug in?

It helps to architect your app in layers and focus on the missing parts before figuring out the glue.

Working on the model is satisfying. It's made up exclusively of things *you* create and control. You can totally immerse and reach a state of flow in your work. No external dependencies, no hassles learning the standard library.

Programming the user interface usually is a totally different kind of task. You will work with Storyboards and set up connections, maybe using segues, maybe using programmatic transitions.

Either way, you're knees deep in Cocoa APIs. And on top of that, you'll have to figure out how to connect this **view layer** to your model layer.

The view controllers want to be fed data. The data is obviously living in the model somewhere, that's what you've created it for.

If you didn't already, you may come up with dedicated entry points to query or mutate data eventually, probably in some kind of Service objects. Maybe your first impulse is to simply fetch data from a Core Data managed object context from within the view controller.

Don't.

{% include figure.md src="/assets/blog/2015/201506191432_mvvm-arch.png" alt="mvvm viper architecture" caption="We can focus on the model first and design view and view model second, then maybe incorporate known concepts like Presenter and Interactor in the Application layer as glue." %}

It's possible to focus on the view logic and design and code the components in relative isolation.

Here's my little trick: don't worry about reaching into the model from your view controllers at all. Isolate it using dedicated data structures.

Your view controllers need data. To ease the mapping of existing model data to the things a view controller consumes, simply start by creating a **view model.** That's either a model _of_ the view or a model _for_ the view, depending on your usage.

If your view is composed of 4 text fields, 1 image, and 1 button, your view model will need 4 `String`s (or `NSString`s) and 1 `UIImage` or `NSImage` properties.

With active view models, you change these data structures to perform visible changes [automatically](http://christiantietze.de/posts/2014/12/mvvm-in-swift/). The model pushes changes to the view.

Using passive view models, your view controller uses these data transfer objects as a convenient means to change the view components. It can populate the fields right away and will not have to worry about rather complicated things like creating a textual representation of an entity.

Even if you know the text field will display a number, don't use a number attribute on your view model -- use a string. Write view models as close to the metal as possible.

All in all, the mapping should not be part of the view. It's part of a layer between your model and the view: **the application layer.** Application services obtain the data and create the view model instances.

If you dipped your toe into using the [VIPER architecture](http://christiantietze.de/posts/2015/03/viper-ios/), you will recognize the Presenter is such a service object: it gets stuff from the Interactor, which lives at the domain end of the application layer, and talks to the user interface. The presenter is at the view end of the spectrum.

