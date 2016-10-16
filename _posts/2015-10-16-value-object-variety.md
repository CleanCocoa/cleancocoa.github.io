---
title: Achieve More with a Variety of Value Objects in Swift
created_at: 2015-10-16 12:04:23 +0200
kind: worklog
tags: [ mvvm, value-object, cqrs, swift ]
image: 201510161203_banana-t.jpg
comments: on
vgwort: http://vg08.met.vgwort.de/na/9928bd0964a947ec8fdda5662964e5da
---

From the department of Domain-Driven Design code patterns, I today present to you: well-named value objects! You can go a lot farther than you have previously imagined with value objects in Swift. Now that Swift is more than a year old, most of us have seen the use of `struct` and heard how useful passing objects by value is.

A typical real-world example is the data-transfer object, which contained a painful amount of boilerplate in Objective-C but became super simple with Swift:

    #!swift
    struct BananaData {
        let size: Int
        let color: UIColor
    }

Suppose we also have a `Banana` entity for which `BananaData` is a simpler representation.

* This data can be used as a [view model][vm] to draw a banana. That's great to decouple the view from the model.
* It can also be the result of parsing JSON from a server, used to construct `Banana` entities later on. That's better than coupling the parser to the model, too.
* When the user can create new bananas, `BananaData` may also be the result of a form the user completes.

You see how versatile `BananaData` is?

Versatility is not always a good sign. It can mean that something is used in the wrong way, too, or that you have missed to model something else because the existing stuff is just enough.

Given how easy it is to create a value object in Swift, let me propose something better than `XYZData`:

    #!swift
    struct NewBanana {
        let size: Int
        let color: UIColor
    }

This is used to create a `Banana` entity only. It took me some getting used to because there's no "Data" suffix to signify that it's not in fact a new banana itself, but data for a new banana. Nowadays, though, its name screams "use me for banana creation" at me.

`NewBanana` should be the result of a user interaction.

"But it just looks like `BananaData` to me! Can't we use it for the JSON parser as well?" -- Yes we could, but then we'd mix banana creation from new data and banana reconstitution (or "hydration", as some call it) from stored data.

To use a `NewBanana` as view model sounds just too plain weird. I can bend my mind and think "display a new banana in the view" means it's novel for the view, as opposed to "update an existing banana view", but that's not working out for me.

You can think of `NewBanana` as a command. If you want to have verbs in your commands, call it `CreateNewBanana`. Why not?

Remember that your project terms may differ and that a "New" prefix doesn't always have to be used for the same concepts.

    #!swift
    struct ParsedBanana {
        let size: Int
        let color: UIColor
    }

Use `ParsedBanana` to signify it's the result of the JSON data. I'd probably stick with `BananaData` here, as "-Data" signifies its a representation of storage contents to me already. That'd make it harder for you to see the result of this change, so I better pick another name. Again, your use of the terms may differ, so consider to use something that's meaningful to _you_, which may be `ParsedBanana`.

    #!swift
    struct BananaViewModel {
        let size: String  // watch this: it's not an Int
        let color: UIColor
    }
    
Use `BananaViewModel` to display bananas in your user interfaces. Notice that the view model sports properties tailored closely for the view: the size will be printed as text, so it's a string. Think "medium" or "2 inches" -- whatever it is, it's prepared for immediate use.

Now you wouldn't put all these banana-related value objects in one place. Instead, put them where they are used: The view model should live in the user interface layer.

The real `Banana` entity should know about none of these, though it's okay for me to extend `Banana` with factories to create a new entity from `ParsedBanana` and `NewBanana`. Although extension affect the type, it seems to be an idiomatic Swift way to create factories and encapsulate knowledge about object creation.

So now our project contains:

* Domain
    * `Banana`, the entity, which is a class, not a struct
* View
    * `BananaViewModel` for display, and 
    * `NewBanana` as a command
* Infrastructure
    * `ParsedBanana`, the result of parsing JSON from disk or server

The three value objects we end up with are strikingly similar. Isn't that a stupid duplication? Well, it _is_ duplication right now. But that's because this is the result of a refactoring towards more meaningful objects.

These diverse types can't be mixed freely, whereas the initial `BananaData` could have come from anywhere and be used everywhere. This enables us to attach specific properties to one of the types without affecting the other uses of banana representation.

For example, add a `date` property to `NewBanana` and store it along the other attributes on your server. Now you have the power to run statistics from the backend, which is another "bounded context" of your application as a whole. Users of your app will not need this statistical information, so we won't include a similar property in `ParsedBanana` or `BananaViewModel`.

This way we end up with different read and write models. That's great news, as we can architect our app differently with more ease to support [CQRS][]. Or use an event store for writing and obtain data in a CRUD way from another store. The all-purpose `BananaData` hasn't made that harder, but it now feels more natural.

Creating low-cost value objects which look similar but _mean_ different things is the way to go to let your app grow. When everything looks the same, it's harder to find leverage to improve code.

---

Photo credit: "[Minion Phil with BANANA!](https://www.flickr.com/photos/wilsonhui/16615215330/)" by Wilson Hui on flickr. License: [CC-BY 2.0](https://creativecommons.org/licenses/by/2.0/)


[vm]: /posts/2015/10/view-model-control/
[cqrs]: http://martinfowler.com/bliki/CQRS.html
