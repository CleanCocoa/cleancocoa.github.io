---
title: "Protocol Madness: Comparing Apples to Oranges from the Vantage Point of Fruit"
created_at: 2015-06-24 08:52:31 +0200
kind: worklog
tags: [ swift, protocol, generics ]
image: 201506240856_fruit-t.jpg
vgwort: http://vg08.met.vgwort.de/na/55cacd6c8b124a73b5f8f337ac5b6935
comments: on
---

Brent Simmons starts with Swift. And just as I, he struggles with value types, protocols, and generics: once a protocol references another or has a dependency on `Self`, you cannot use it as a type constraint or placeholder for all descending objects. You either have to use generics to expose a variant for each concrete type, or you re-write the protocol. There's no equivalent to the way Objective-C did this: something like `id<TheProtocol>` just doesn't exist. Swift is stricter. So how do you deal with that?

In his [original post](http://inessential.com/2015/06/21/swift_protocols_question), Brent ends up with this:

    #!swift
    protocol Value: Equatable { }
    
    protocol Smashable {
      // Use generics because `Equatable` depends on `Self`
      func valueBySmashing​OtherValue​<T>(value: T) -> T
    }

Now a concrete implementation is only allowed to return type `T`. Say you've got `Apple`s and `Oranges` both implementing `Value`, then you cannot simply return an `Apple` from `valueBySmashingOtherValue(_:)`:

    #!swift
    struct Apple: Value {}
    struct Orange: Value {}
    struct AppleSmasher {
      func valueBySmashingOtherValue<T>(value: T) -> T {
        return Apple() // Doesn't compile!
      }
    }

That's because with the dependency on `Equatable`, `valueBySmashingOtherValue(_:)` _kind of_ works as if there were two concrete alternate method implementations, one for `Apple`s, one for `Oranges` (and one for every other `Value`). That's not how Swift does it internally, but it's a good mental model.

Andy Matuschak [proposed](https://gist.github.com/andymatuschak/a40c4c699c0abdd704ce) using an enum when there's only a fixed amount of implementations involved. That's what Brent is [discussing](http://inessential.com/2015/06/23/swift_question_enum_vs_struct_for_vari) now:

    #!swift
    enum Value {
      case variantApple(Apple)
      case variantOrange(Orange)
      // ...
    }

If there's logic which applies to some but not all cases of this enum, you have to write switch statements to filter allowed cases.

This is how filtering could be done, and what Brent currently seems to have in mind:

    #!swift
    extension Value {
      func canCombineWith(other: Value) -> Bool { /* ... */ }
      func valueByCombiningWith(other: Value) -> Value { /* ... */ }
    }

I suppose he's using the boolean function as a check in the combination function. If the check fails, what then? Return `self`?

{% include figure.md src="/assets/blog/2015/201506240856_fruit.jpg" alt="fruits" caption="Photo credit: <a href=\"https://www.flickr.com/photos/nubobo/8226113305/\">りんご、オレンジ</a> by <a href=\"https://www.flickr.com/photos/nubobo/\">nunobo</a>. License: <a href=\"https://creativecommons.org/licenses/by/2.0/\">CC-BY</a>" %}

I would prefer it the compiler helped. That's what the inflexible generics approach does. Remember it's not like "treat this object just like a `Value` and ignore its specific capabilities", but a much stronger claim if it has certain properties like depending on `Self` or using associated types.

    #!swift
    protocol Fruit: Equatable { }
    struct Apple: Fruit {}
    struct Orange: Fruit {}
    
    protocol Saucable {

        func sauceWith(evenMoreOf: Self)
    }

    struct Saucer {
    
        func sauce<T: protocol<Fruit, Saucable>>(one: T, other: T) {
            one.sauceWith(other)
        }
    }

    extension Apple: Saucable {
    
        func sauceWith(evenMoreOf: Apple) {
            // do something cool
        }
    }

Here, I've used an anonymous combined protocol: `protocol<Value, Saucable>`. Currently, this will only apply to Apples. Because Oranges have to be peeled. Let's add this, too.

    #!swift
    protocol Peelable {
        typealias FruitPult: Fruit
    
        func peel() -> FruitPult
    }

    struct OrangePulp: Fruit { }

    extension Orange: Peelable {
    
        // For this type, `FruitPulp` == `OrangePulp`
        func peel() -> OrangePulp { 
            return OrangePulp()
        }
    }

    extension OrangePulp: Saucable {
    
        func sauceWith(evenMoreOf: OrangePulp) {
            // mushed orange?
        }
    }

I just made it so we end up with another case of being forced into generics if we wanted to work with `Peelable`: it has an associated type, which can vary and is even more complex than dependency on `Self`.

For brevity, let's make `Orange` more intelligent:

    #!swift
    extension Orange: Saucable {
    
        func sauceWith(other: Orange) {
            peel().sauceWith(other.peel())
        }
    }

    let saucer = Saucer()
    saucer.sauce(Orange(), other: Orange())

It still won't work to mix stuff:

    saucer.sauce(Apple(), other: Orange()) // Doesn't compile!

In some cases, this is very much intended. What should adding arrays to strings do, for example? 

If you want it to compile, you need a more flexible `Saucer`:

    #!swift
    protocol Saucable {
    
        func sauceWith<T: Saucable>(evenMoreOf: T)
    }

    struct Saucer {
    
        func sauce<T: protocol<Fruit, Saucable>, U: protocol<Fruit, Saucable>>(one: T, other: U) {
            one.sauceWith(other)
        }
    }

    extension Apple: Saucable {
    
        func sauceWith<T: Saucable>(evenMoreOf: T) { }
    }
    
    extension OrangePulp: Saucable {
    
        func sauceWith<T: Saucable>(evenMoreOf: T) { }
    }

Now two different `Saucable`s can be combined whereas previously they had to be the same kind. 

`Orange` cannot depend the other ingredient to be an `Orange` and call `peel()` on it. Thus we add another level of indirection, namely double-dispatch, but this time with the same method:

    #!swift
    extension Orange: Saucable {
    
        func sauceWith<T : Saucable>(evenMoreOf: T) {
            evenMoreOf.sauceWith(peel())
        }
    }

With two `Orange`s, A and B, we get the following call stack:

* `A.sauceWith(B)`
* `B.sauceWith(OrangePulpOfA)`
* `OrangePulpOfA.sauceWith(OrangePulpOfB)`

With mixed Fruit, where B is an `Apple`:

* `A.sauceWith(B)`
* `B.sauceWith(OrangePulpOfA)`

Works.

## Applied to Brent's Sample Code

Brent wants to work with about a dozen different kinds of `Value`. Some can be added (numbers), some can be coerced (strings):

    #!swift
    func valueByAddingValue​(value: Value) -> Value
    func valueByCoercingToType​(type: Value) -> Value

With the different kinds of power of generics we've just witnessed, that should instead be modeled as:

    #!swift
    protocol Value: Equatable {}
    
    protocol Addable {
        func add(other: Self)
    }
    
    protocol Coercible {
        func coerce(with: Self)
    }
    
    struct IntValue: Value, Addable {
    
        let value: Int
    
        func add(other: IntValue) -> IntValue {
        
            let result = value + other.value
            return IntValue(value: result)
        }
    }
    
    func ==(lhs: IntValue, rhs: IntValue) -> Bool {
    
        return lhs.value == rhs.value
    }

And if in need for a higher-level function:

    #!swift
    func valueByAddingValue​s<T: protocol<Value, Addable>>(one: T, other: T) -> T {
    
        return one.add(other)
    }

This will work on a pair of equal types. That's great, because now integers can be added to one another and strings can be added to one another, but no integers can be added to strings.

If you want to mix and add `IntValue`s to `DoubleValue`s, say, you should re-write `Addable` to require something like `add<T: Addable>(other: T) -> Self`. It's important to decide which return type to use in that case: should adding a float to an IntType result in another `IntType` or another `FloatType`? Can't have both. (That's not how we think about addition, which should be commutative: `1 + 2 = 2 + 1`.)

## Conclusion

Thanks to Swift's strong typing, I spent a day or two figuring out how to model value objects in my own apps. I end up not requiring protocols to be `Equatable` a lot, but instead make the implementations `Equatable`. Or I supply the `==(_:,_:)` method anyway without the protocol dependency. It works, too.

Protocol-based programming to me means: using a lot of tiny protocols and combine them into value types. Brent's `Value` protocol probably started too big, doing too much. You can almost always add another level of abstraction, refactor types into another protocol, and put behavior into yet another, more generalized function.

The fruit example was a bit contrived from a user's perspective. But I think it demonstrates really well what's going on when you mix types.

**<%= rel_link_to "Download the Playground (Swift 1.2)", "AppleOranges.zip" %>**
