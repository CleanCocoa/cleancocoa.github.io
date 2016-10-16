---
title: Extract Private Functions as Collaborators
created_at: 2016-02-05 15:31:29 +0100
kind: worklog
tags: [ testing, oop, srp, tdd ]
vgwort: http://vg01.met.vgwort.de/na/eac0ca5a5b304f3491aa8a114bb59e85
comments: on
---

> "Using a private function means having a hardwired link to an anonymous collaborator. Over time, this will slowly hurt more." ([@jbrains][tweet])

[tweet]: https://twitter.com/jbrains/status/687683605825515520

I was thinking about this the other day when I wrote tests for a presenter. It receives a date, formats it, and makes the view update a label with the result. Simple. (The real thing actually does a bit more, but that doesn't matter much.)

The presenter did this as part of the information flow when the user selects a new date, `showDate(date: NSDate)`.

Now to populate initial view state, the presenter should also show today's date first: `showToday()`. Instincively, I pulled the repeated code into a private method, here shown in a contrived way for the sake of example code:[^fn]

    #!swift
    func showToday() {
        let text = formatDate(NSDate())
        showDateText(text)
    }
    
    func showDate(date: NSDate) {
        let text = formatDate(date)
        showDateText(text)
    }
    
    private func showDateText(text: String) {
        view.displayDate(text)
    }

[^fn]: It's contrived because I could just as well delegate from `showToday` to `showDate` with the same effect, only then you will not see what's bad about it as clearly.

The unit tests revealed that there are now two places, two events which result in a similar view change. Not too much trouble at first, but add some complex calculations and showing a loading indicator to the mix and you'll end up with quite a lot of duplicate tests.

**Duplicate code is easy to produce and doesn't hurt much when you write it. But testing duplicate code, thereby producing duplicate tests, feels outright stupefying.** That's the design feedback unit tests will give you. In this case the tests told me I'm being stupid. (Thanks.)

I remembered J. B. Rainsberger's [tweet][] from above. The "private" keyword may be leading me to something. What kind of collaborator is hidden here?

In fact, it's a ... *presenter*! -- Wait, what? Didn't I write that one just now? 

The stuff the presenter deals with is hidden in the private `showDateText` method. So I extracted the varying parts (the inputs or callers of this method), `showToday` and `showDate`, into an `EventHandler` (or commander, or whatever) which delegates to the presenter. `showDateText` now is the public API of the presenter while the two different triggers for showing a date text are now encapsulated in another objects.

    #!swift
    class EventHandler {
        let presenter: Presenter
        
        init(presenter: Presenter) {
            self.presenter = presenter
        }
        
        func showToday() {
            presenter.showDate(NSDate())
        }
    
        func showSpecificDate(date: NSDate) {
            presenter.showDate(date)
        }
    }

    protocol View {
        func displayDate(text: String)
    }
    
    class Presenter {
        let view: View
        
        init(view: View) {
            self.view = view
        }
        
        func showDate(date: NSDate) {
            let text = formatDate(date)
            view.displayDate(text)
        }
    }
    
    
Coincidentally, this is the problem I usually have with implementations of the VIPER pattern: sample projects use a `XYZPresenter` for both issuing view changes and reacting to view events. This muddies the single responsibility of the presenter: to maka data presentable and tell the view what to do.
