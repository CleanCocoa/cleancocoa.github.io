---
title: "Configuration Objects: Delegate Initialization to a Parameter"
created_at: 2015-11-28 16:15:53 +0100
kind: worklog
tags: [ east-oriented, initialization ]
comments: on
---

The [Builder design pattern][builder] is often overlooked, I find. Apart from plain builders, in Ruby I found that some us it like a configurator. You can use a configuration object in Swift in the for of block parameters, for example:

    #!swift
    thing = ComplexThing() { config in
        config.boringProperty = 123
        config.addThing(foo)
    }
    
This would make the internals of `ComplexThing` configuratable through the block. Instead of messing around with an instance of a `ComplexThing`, you can do atuff with the configurator which determines how the instance will look.

    #!swift
    class ComplexThing {
        
        class Builder {
            var boringProperty: Int
            var interestingList = [OtherThing]()
            
            func addThing(thing: OtherThing) {
                interestingList.append(thing)
            }
        }
        
        let property: Int
        let things: [OtherThing]

        init(config: (ComplexThing.Builder) -> Void) {
            
            let builder = ComplexThing.Builder()
            config(builder)
            
            self.property = builder.boringProperty
            self.things = builder.interestingList
        }
    }

You see, the utility of this approach in my day-to-day life is not very clear, yet, or I'd have come up with a better example.

Say you have 2+ server configurations and would like to setup a connection according to these -- why shouldn't you store these configurations in a value object from which the connection reads and populates its attributes? The only difference is querying for values VS setting values.

I am not a friend of dogmatism, so even though I emphasize the utility of [East-Oriented Programming][east], you shouldn't just use this pattern because it's "East" while making things harder to read.

Will post an update if I find a good real-world use case.

[east]: /posts/tags/east-oriented/
[builder]:  https://en.wikipedia.org/wiki/Builder_pattern
[param]: /posts/2015/11/parameter-objects-simplify/
