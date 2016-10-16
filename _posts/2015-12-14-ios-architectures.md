---
title: iOS View Architectures and VIPER Explained
created_at: 2015-12-14 11:00:41 +0100
kind: worklog
tags: [ viper, ios, software-architecture, mvvm, mvc, cqrs ]
preview: fulltext
comments: on
---

There's an excellent [overview of MVC, MVP, MVVM, and VIPER in iOS][arch] with sample codes by Bohdan Orlov of Badoo I recommend you read.

There are two main takeaways:

1. One of the commenters nailed it: any MV* variant focuses on architecting view components. Only VIPER takes the application as a whole into account. MVC by itself is an architectural pattern, but not an approach to app architecture in itself.
2. A `UIViewController` belongs into the user interface layer and is, in fact, a composite view itself. That's confusing at first because of the name. This insight will liberate you from thinking that a view controller is sufficient to glue data to UIKit components. There's room for a whole application between these two.

The VIPER example is exceptionally good. It takes VIPER's heritage of _Clean Architecture_ and _Hexagonal_ into account and defines the Interactor through an output port. In that way Bohdan's sample code is more east-oriented and cleaner than what you'd usually find on the topic:

    #!swift

    protocol GreetingProvider {
        func provideGreetingData()
    }

    protocol GreetingOutput: class {
        func receiveGreetingData(greetingData: GreetingData)
    }

    class GreetingInteractor : GreetingProvider {
        weak var output: GreetingOutput!
    
        func provideGreetingData() {
            let person = Person(firstName: "David", lastName: "Blaine") // usually comes from data access layer
            let subject = person.firstName + " " + person.lastName
            let greeting = GreetingData(greeting: "Hello", subject: subject)
            self.output.receiveGreetingData(greeting)
        }
    }

Usually, you'd model `provideGreetingData()` as a function that returns the data to the caller. This will cause trouble in async processes of course. 

You see in [the full example](https://gist.github.com/BohdanOrlov/ae3158cad9b7e75a8099/) that the amount of types seem to explode. Don't be afraid of that as long as you can assign each type a specific responsibility. Then it won't turn into the mess everyone seems to be afraid of.

Having used VIPER in apps myself, I see a problem with the names, though. `XYZInteractor` and `XYZPresenter` aren't much better than `XYZController` in terms of expressiveness. On top of that, a concrete Presenter is always modelled to act as event handler, too. Don't let this fool you into thinking you absolutely _have_ to do this yourself -- there's always room to separate writing from reading operations, or event handling from view population.

[arch]: https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52
