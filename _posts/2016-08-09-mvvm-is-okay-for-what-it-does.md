---
title: MVVM Is Quite Okay at What It Is Supposed to Do
created_at: 2016-08-09 13:32:11 +0200
kind: worklog
tags: [ mvvm, software-architecture, viper, tdd, layered-architecture, srp, clean, model ]
vgwort: http://vg01.met.vgwort.de/na/1038894321f941c6a7a19816392807d7
comments: on
toc: on
---

Criticism targeting MVVM (Model--View--View-Model) from late last year essentially points out it's not the silver bullet some take it for.[^delay] Most of the stuff is missing the point. How are you supposed to make sense of it? What's good advice for _your_ project?

I want to raise awareness about the underlying issues that often go undiscussed: the use and limits of patterns like MVVM, and understanding the problems you try to solve with it.

[^delay]: I wrote most of this post in December 2015. Now it's August. I wonder where the time went ...

## Critical MVVM: What's needed is a change of structure -- whatever that means

MVVM, as it's practiced, has a few problems. I linked to [Soroush's post "MVVM Is Not Very Good"](http://khanlou.com/2015/12/mvvm-is-not-very-good/) in an aside before Christmas. His post's title says it all already. Then I found [Ash Furrow react](https://ashfurrow.com/blog/mvvm-is-exceptionally-ok) to Soroush's criticism of the pattern, "MVVM Is Exceptionally OK", trying to find a common ground of critique. 

Ash's reconstruction of the pain points is this:

> 1. MVVM is poorly-named.
> 2. MVVM invites many responsibilities.
> 3. MVVM doesn’t change your structure.

Both Soroush and Ash agree that MVVM is poorly named. And it can move the problem of massive view controllers to "massive view models," if you will.

The second point is linked to the first. When you move everything into a view model, it simply becomes a garbage can. It does lots of things in different contexts.

The third point is of real interest. Both Ash and Soroush want to change the very structure of code when they encounter massive view controllers. Jason of [NearTheSpeedOfLight.com](http://nearthespeedoflight.com/article/2015_12_21_mvvm_is_not_good_and_not_mvvm_is_good) phrases it thus: "We need better ways of organizing our code." So if MVVM fails at this point, it means MVVM doesn't provide a better structure.

People seem to expect a change of structure, a new way to organize code, but MVVM hasn't met their requirements. 

"But I just created a view model type that wasn't there before," you call out. Sure creating a type equals a change of structure, doesn't it? It's a new object type, after all! -- We'll have a look at this and find out what "structure" entails and why this intuition is wrong.

Secondly, if a change of structure is still needed, which problem is actually solved by applying MVVM? Why do people expect this in the first place, and how does such an improvement come about instead?


## Does your app have a structure, and if so what does it look like?

A "change of structure" implies there is a structure in Cocoa apps for both Mac and iOS. Because we can deviate from traditional MVC to MVVM or whatever, the structure does not equal MVC. If MVC defined the structure, MVVM would've changed it. Also, massive view controllers could then be a feature, not a burden.

Apple's documentation gives no additional guidance in terms of structuring your code. If we rule out MVC, there's nothing left, it seems.

We need to come up with an answer to this on our own. (Maybe rightly so, because every app is different after all.)

Pondering the term "structure," we can probably agree on a very abstract level that it's something that has relationships between elements. Some things are foundational, others build on top of them. Some things are essential, others are accidental, nonessential. We have to find out what kind of elements we're talking about to make this more concrete.

If we knew what this mysterious structure is, we could design it. Unless we're aware of our needs in terms of designing structures, it's likely that a structure will simply emerge from your code. According to our rough definition from above, there can be no software application without structure. (Except maybe when your app consists of an infinite loop which prints a string to the console.)

On iOS, you're going to have a blank project with an `AppDelegate` and a storyboard plus `UIViewController` subclass. These things have a relation with one another. That's how Apple provides guidance for structuring our apps. If your app is at least this complex, you have structure. And it grows. There's no way around that. 

The question really is this: do you consciously influence the way your application's structure changes?


## Getting a grip on structure with layers

In 2015 I've discussed a few example codes by other programmers. I liked what they did and tried to improve the usefulness of their examples through pointing out the **seams** that show how an app can include the new functionality from that.

For example:

- [Achieve More with a Variety of Value Objects in Swift](/posts/2015/10/value-object-variety/)
- [Decouple UI from Model with View Models and Controls](/posts/2015/10/view-model-control/)

I took isolated examples which show how a technique works and embedded it into something bigger. In other words, I did part of the hard work for you. I took _the burden of integrating_ a pattern into an app. Each of us has to figure that to utilize something new: where does this code go? Is it a new "thing"? If it's a thing, how do other things interact with it? These are the seams in your code.

Through that practice I let a framework to analyze the role of components emerge. It's based on my understanding of [layered architecture][layered] which is super simple but helps tremendously to divide the world into cohesive units or "modules." 

Layers help to show where particular seams are missing in particularly complex code samples. These are code samples which are simplified to bring the point home while they fail to show how you can integrate the functionality into your software. I find this is the stuff that's often causing headaches. Especially beginners don't have the analytic experience to decompose sample code and transmute it into their own apps.

The layers I talked about in these posts were:

- (Domain) **Model**, which is kind of self-explanatory: the entities your application is about. There are subtle nuances you can take into account but we don't  worry about that right now.
- **Application** logic; there reside the use cases of the software, the service objects that glue stuff together. Here's the algorithm which pulls data from a web service and stores it locally.
- **Infrastructure**, which contains persistence mechanisms and networking code. Not the actual calls, but the types of URL handlers and database accessors or whatever you got.
- **User Interface** a.k.a. the View. Stuff displayed on screen. `UIViewControllers` are part of this. (We can argue about that, but let's just use this prescription for the sake of this article.)

This is not *The Truth*, mind you. It's just one admittedly simple way to structure an application code base. It's a tool to think about your application on a high level that goes way beyond "class" and "method." Simple apps may not need to distinguish between *Application* and *Infrastructure*; some complex applications need to subdivide *Infrastructure* further.

Think about a messy project from your past. If it suffered from massive view controller syndrome, thinking in terms of layers makes it hard to locate the view controller anywhere. Parts of it belong to networking, parts clearly belong to the view, parts deal with persisting data. Refactoring that massive view controller into types which clearly belong into one of these layers can help -- help to extend the functionality further and to replace framework dependencies (switching from Parse to Realm), for example.


## Mistaking MVVM for a software architecture approach is the real problem

Now MVVM is mostly concerned with the Application and UI layer: a "real" model is supposed to exist already and you're supposed to somehow produce a "view model". 

MVVM's proposes that a view model will be passed to view components. How? Nobody tells.

Since MVVM introductions don't talk about this in-depth a lot, I guess most people will put the logic into a view controller out of habit. The view controller is part of the User Interface layer, though, and thus the real model enters the UI layer through that view controller. I argue that the view model is part of the UI layer and that no component in this layer should be coupled to the real model for the sake of encapsulation. 

Here we have a conflict: the real model enters the UI layer but it shouldn't. It's an easy problem: create the view model somewhere else and pass it to the view controller instead. But where is this other place? (Why do articles about MVVM not talk about this stuff?)

**The narrow focus on a pattern that doesn't take most of the application into account is the number one source of problems.** 

It's a confusion of concerns. Articles about MVVM do not help with this because that is beyond their scope. MVVM intros I know about don't tell you how to craft an app and use MVVM as a way to structure the UI layer. Structural concerns are part of "crafting an app," a different skill than "implementing a pattern."

MVVM focuses on solving problems in code. Not every kind of problem but those of writing user interface code. **MVVM is a pattern of the solution space, that is your code.** Architecture tackles the problem space: driving a car remotely, taking selfies, catching "pocket monsters", you name it. You can divide the problem space into things (car, engine, speed, size, grip) which you model in code (`class Car`, `struct CarSize`). Thus you get from problem space to solution space. MVVM is all about code. An architecture focuses on the plan to create a software solution. Design patterns are best practices. The "design" in "design pattern" is not about software design -- it's about designing code of components.

What the creation of software looks like, simplified:

1. Articulate a problem that needs a solution
2. Analyze the problem
3. Model the problem (in code)
4. Create an app of that

How it's done in the real world:

1. Articulate a problem
2. Create an Xcode project and start building the interface
3. Search for patterns on Stack Overflow

[layered]: https://en.wikipedia.org/wiki/Multilayered_architecture

So here's part of the reason why MVVM doesn't seem to fit or solve everything it was advertised to do: it is rooted in a coding mindset which is not what you need to change (or come up with) a structure. 

## Layered architecture can help come up with a better understanding of the problems you have

I proposed layered architecture because it's so simple and yet effective. Its focus on the application's structure or architecture goes way beyond the scope MVVM can have. Understanding the difference between design patterns and software architecture is important to employ either.

The proverbial problem with having hammer and nails as your only tools is apparent most when you're actually supposed to be painting a picture or baking a cake. 

Now people give you a larger toolbox with screwdrivers and all, because they say the MVC hammer doesn't suffice -- but until you understand which tool is good for what you still won't know why the larger toolbox doesn't help.

Thinking in layers is all about drawing boundaries between larger-scale parts of your application.
This reasoning can be employed to plan the next component's design in advance. 

Drawing boundaries inside your app makes a huge difference. Massive view controller syndrome is a synonym for "no boundaries, everything in one object"-design. MVVM helps to break up these mess across object responsibilities: you recognize the responsibility of a view model and put it into its own type. The next step to extract functionality out of the view controller is not obvious, though, if all you know is MVVM.

Say your view controller accidentally includes database code. The layers I listed above can already help to find out that there's another layer next to the UI layer where you can move it.
 
Talking about even this very basic way to architect is so uncommon in the Cocoa community that I really wonder how teams get stuff done, like, *at all.*  If nobody ever talks about any form of higher-level structures in apps, it's reasonable to assume there is none. That's kind of troubling: when we cannot talk about the design of an application at large, we're left with solving micro problems, with implementing features. **When there's a lack of design, we're inapt to improve it.**

## VIPER is a different phenomenon because it's not just a design pattern

VIPER, too, became a popular topic last year (totally subjective). I guess for some people, especially beginners, there's a choice to make: adopt VIPER or MVVM, aka "which trend  should I follow?"

The dichotomy is misleading. 

VIPER takes much more of an app's structure into account than the design pattern MVVM can. That's not a coincidence: **VIPER is a software architectural style, not a design pattern.**

VIPER offers a systematic approach to creating components with clear responsibilities. It comes as a framework to analyze a complete slice of a working application: from fetching data (Interactor) to passing it to the view (Presenter) and reacting to user input (Event Handler) and wiring all of them together (Wireframe). It promises peace of mind during development through reducing the likelihood that you connect objects in a way that's not beneficial to your app. MVVM only promises to keep the view dumb.

Of course there are people who struggle to apply the principles of VIPER. Creating all the components of VIPER advocates seems to introduce bloat first and foremost: you have wireframes, presenters, interactors, the view, and model entities -- "so many classes!"

What's wrong with more types? I don't say that more is better, but less isn't better, either, if you have no reason. 

If you hesitate to create new types just "because", you end up stuffing everything into a view controller. Bloat is bad, sure. But proper separation of concerns among objects requires more than just a single object. 

The feeling that VIPER is bloated may just as well stem from the fact that creating 5 objects to show and handle a single view controller feels ... weird. If you never worked that way, that'll be uncomfortable. Maybe that's all of it.

Trust VIPER and see where it leads. The experience can be very educating. But it's risky to follow the rules without knowing the reason they exist. Using VIPER works better when you're experienced with designing structures.

One first has to learn to put event handling into one component while view setup is handled elsewhere. This is different from adding functionality you need just where you need it. That's mentally taxing at first. It's a challenge. You'll be a better programmer when you learn to understand what's happening there, but it's still not feeling nice. (It's the same with all kinds of training in life, really.) 

## Knowledge is the true bottleneck

Once you have a better grip of software architecture, you're able to adapt VIPER more creatively. 

You don't need magic or a special gift to come up with an architectural pattern like VIPER. You need knowledge and experience and creativity. When you can come up with something on your own, you can interpret the rules of VIPER, layered architecture, or whatever else you find on the web properly. You can pick up new ideas and come up with solutions on your own instead of looking for a fitting solution out there.

Following patterns slavishly just because others tell you it's working for them will only change the kind of problem you have. You'll introduce knowledge debt into your application. It's like technical debt, like introducing dependencies you don't understand properly, only on a more general level. 

With a better understanding you can utilize MVC or MVVM where appropriate. And you'll understand that VIPER solves a different kind of problem than MVVM, and that VIPER + MVVM totally makes sense: the Presenter got data from an Interactor and prepares a View Model instance which it passes to the View. That's may not be part of a book on VIPER, but it's a compatible design.

The underlying inaptitude to reason about application design and architecture is one of the root causes of confusion.

Constructing a simple "about this app" screen with all 5 components of VIPER is madness. Try it; a lot of the components will turn out to be wasteful and devoid of much content. Showing static text doesn't call for Presenters, Interactors, and input/output ports.

Every plan is bound to be poorly executed when those that execute it don't understand what's going on. They cannot react to changes in the environment.

VIPER is such a plan. It's an approach to architect iOS apps. Without proper understanding of VIPER's internal logic it's very hard to apply it successfully.


## "Coincidentally", testing your code makes it easier to recombine components later and come up with new architectures

Here's actionable advice: 

**Don't force any architectural patterns down your throat.**

Maybe your application won't benefit from a particular architectural approach. Maybe your app is too simple or too complex for the approach you have in mind.

Trying to spring-clean existing code by forcing it into the constraints of VIPER, for example, can actually harm the project.

How can you find out what's appropriate? What's the measurement?

Here's the single best thing I encountered so far: **Strive for higher testability first.** (I know, I know. Bear with me for a minute.)

Tests will help you get to new levels all on your own -- levels where smarter people than us went before. Levels from which these people derived principles and architectural approaches. 

Testing will teach you to think and reason about your code all the time. It _forces_ you to create components that can be called from tests with ease and which sport predictable behavior. If your components can be tested in isolation, you're free to re-combine them and let order emerge, bottom-up instead of top-down.

Most design patterns and higher level architectures make your code easier to test. That's not a coincidence. That's a reflection of the coming-about of the pattern.

## How I distinguish the model from a view model in TableFlip

Working on [TableFlip](http://tableflipapp.com) uncovered that keeping tables as columns and rows in memory doesn't work when you want to create cells that span multiple columns. A naive approach of a 2-dimensional array is misleading. The atom of a table, the cell, is more fundamental. Columns and rows are just ways to look at the cell-based data. 

TableFlip's model has a `Table` which owns many `Cell`s and cooperates with `ColumnLens`es and `RowLens`es to provide different ways to enumerate the data. I didn't stuff a column or row based view to the data into `Table`. Test feedback drove me to separate table from columns and rows: the lenses' tests don't care about `Table` internals; they only expect input data in a certain way. That's more comfortable to handle in tests. And it makes the resulting objects more cohesive and independent. High internal cohesion is very helpful to keep your overall code readable. (See [CLEAN code](/posts/2015/09/clean-code/).)

All of this core modeling is highly view agnostic. Not only does the view have no clue about the model, the model doesn't care about the view, either. 

I have the core model to make changing state as easy as possible. "Command-line interface"-easy. It can be saved, viewed, deleted -- you don't "save" the user interface. 

## Testing frees your mind to do creative stuff

"Is this part working? Lemme run the app and see ..." -- manual testing of the whole app gets tedious if the state gets more complex. Automated unit testing provides similar feedback ("Is that part working? Lemme use the class and check if the returned data is what I expected ...") only faster, more isolated -- and without the manual operation.

TableFlip currently has 300 tests, most of them in the model layer (I didn't write tests for most of the UI). With a single keystroke I can verify that 300 things work as expected. Manual testing doesn't scale.

Another obvious downside of manual testing: Refactoring is too risky. Fear of regression, fear of breaking code again is what hinders all of us to make progress. Even the most obvious piece of ugly code can become hard to improve if you don't know whether a change will break the app.

Unit testing is annoying a lot of times. And that's precisely what it's good for. It tell you when things become hard to control, observe, and change. On the upside, when testable components become the foundation of your software, you will recognize classes of collaborators. 

Simple UIKit or AppKit-based apps take a couple of minutes to prototype in Interface Builder. We're rewarded with walking skeletons of apps quickly. Tapping a button performs a visible action. Writing tests for model code is like navigating your Mac from a Terminal. Less fun. Less obvious to do.

But keep in mind that you don't code an app in Storyboards alone. Glue code begins to appear: fetching data from Core Data takes code. Where will it go? Well, just put it into the view controller for starters and see what happens. -- And thus the degradation of code quality begins.

## Stop focusing on the wrong end: UIKit code

If you write unit tests early, you won't tolerate these small quick fixes and cheats all the time. Because tests will become hard to write and maintain. And you would have to re-write production code to make it testable.

Bottom line of all this:

**Your app is not about UIKit.**

It isn't about Core Data either. Or Parse. Or AlamoFire. Or ReactiveCocoa.

Neither is your app a folder full of swift files that contain structs which encapsulate business logic. This might be a nice model, like a painting is a nice depiction of the scene before your eyes, but it's not a piece of software. 

Your app is the whole running program with user-facing components. It solves problems of its users. That entails some input and some output. And a flow of information in between. 

The focus was on the problem space all the time. We just didn't notice.
