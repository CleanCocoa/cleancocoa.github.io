---
title: Thinking in Terms of Functional Programming Encourages Clean Factoring of Code
created_at: 2015-02-04 16:43:16 +0100
kind: worklog
tags: [ functional-programming, swift ]
vgwort: http://vg08.met.vgwort.de/na/b6434d21e8804802aa2969ca3c44e433
comments: on
---

Chris Eidhof provided an excellent example for [creating a mini Swift networking library][ce]. It executes simple requests and can be extended to work with JSON (or XML, if you must) without any change. The [accompanying talk](http://realm.io/news/chris-eidhof-micro-libraries-swift/) is worth watching, too. You should check it out to see well-factored Swift code in action. It's a great example.

I want to comment on a part near the end here:

> For me, this is the power of functional programming. We wrote one `apiRequest` function and one Resource datatype, and never changed it again. We can provide this in a library. We then created a couple of small functions for encoding and decoding JSON, and wrapped the existing functions to create our own convenience functions. The `authenticatedUser` is now very tiny.

Where `authenticatedUser` is:

    func parseGithubProfile(dict: JSONDictionary) -> GithubProfile? {
        return curry(makeGithubProfile) <*> string(dict, "login")
                                        <*> int(dict, "id")
                                        <*> url(dict, "avatar_url")
    }

    func authenticatedUser() -> Resource<GithubProfile> {
        return jsonResource("user", .GET, [:], parseGithubProfile)
    }

Read the whole code [on GitHub](https://gist.github.com/chriseidhof/26bda788f13b3e8a279c).

Chris achieved **well-factored code**: each method's intent is crystal clear. They are short and read well (if you know the custom operators like `<*>` and `>>=`, that is). But is it due to the power of functional programming, really? I want to argue that this isn't the case.

Unlike Haskell, for example, Swift isn't a purely functional programming language. But a lot of the functional programming paradigms can be applied to working with Swift code.

Functions are [first-class citizens](http://en.wikipedia.org/wiki/First-class_citizen) in Swift. So you can say that functional programming is indeed possible. Java, on the other hand, wouldn't support functional programming: closures in Java are wrapped in objects still. It's just syntactic sugar in Java-land. Swift treats closures like other values, objects, or entities.

Take a look at this line of code:

    return jsonResource("user", .GET, [:], parseGithubProfile)

It returns the return value of a function. The last parameter is a pointer to another function, `parseGithubProfile`, which is defined in the code above. `jsonResource(...)` allows you to pass in a closure to do the actual handling of a return value. This makes the function very efficient to write.

With traditional object-oriented programming, you had to pass in a handler _object_ instead of a function pointer or closure. Let's say the OOP adaptation would look something like this:

    class GithubProfileParser : JSONParser {
        func parse(data: JSONDictionary) -> Resource<GithubProfile> { /* ... */ }
    }
    
    func authenticatedUser() -> Resource<GithubProfile> {
        return jsonResource("user", .GET, [:], parseGithubProfile)
    }

The drawbacks are imminent: you have to be a lot more explicit. You can easily declare a function or pass in an anonymous closure. It takes code to declare a class and the protocols it has to adhere to.

Chris' approach to `jsonResource`, on the other hand, only has to make the notation of the closure explicit -- accept a `JSONDictionary` parameter and return something, expressed as: `JSONDictionary -> A?`, whereas the generic `A` depends on the return value of `jsonResource` itself. This is a lot more dense than something like:

    protocol ParsesJSON {
        func parse<A>(data: JSONDictionary) -> A?
    }

Functional programming deserves credit for making Swift code short. It doesn't deserve credit for making the code better by itself, because the same adaptability should be the aim of OOP, too. Only with higher cost.

Take a look at Chris' book [_Functional Programming in Swift_](http://www.objc.io/books/) to learn more about the shortcuts you can take. It's worth it.

[ce]: http://chris.eidhof.nl/posts/tiny-networking-in-swift.html
