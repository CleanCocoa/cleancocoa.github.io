---
title: Combining Collection-Like Repositories and East-Oriented Code
created_at: 2015-05-07 09:10:39 +0200
updated_at: 2015-07-09 12:54:00 +0200
kind: worklog
tags: [ repository, east-oriented ]
vgwort: http://vg08.met.vgwort.de/na/0529e91516ca46df9d63b65e79c82c4c
comments: on
code: swift
---


A repository pattern is used to model a central place in your domain to fetch model instances. It usually hides database-related stuff behind a collection-like interface. You don't have to worry about caching or database query optimization in client code -- the concrete repository implementation will handle that.

East-Oriented Code advises us to write methods so that <del>they perform side-effects but not return values</del> <ins>you can chain calls</ins>. This way, information never travel west, from right to left. Here's a common example:

    #!swift
    performSomeAction() // Information travels in reading direction ("east")
    let result = computeSomething() // Return value lands in the variable
                                    // to the left ("west").

To chain calls, methods should return `self`. We can chain calls this way:

    #!swift
    actionator.doSomething().andSomethingElse()

Since the repository is a collection, it _returns_ stuff. Most of its interactions are traditionally query-based, like `allCookies() -> [Cookie]` or similar. Here, we want to obtain a set of cookies, so it makes sense to write a query method instead of a command.

Let's bend our minds a bit and try to imagine if there's a feasible way to model repositories in east-oriented fashion:

* queries should not return the result of the operation but a variant of `self`
* this variant of `self` could be a modified version, but that doesn't make much sense for repositories, so
* return the same repository instance and push the result "to the right" through some callback mechanism.

There's always the option to supply handler blocks. I talked about these in respect to [error handling](http://christiantietze.de/posts/2015/02/functional-swift-exceptions/) already.

If we model our repository to return `self` and take a collaborating handler object to deal with the results, we may end up with a chain-able example like the following:

    #!swift
    cookieRepo.eatCookie(aCookie).allCookies(handler)

Keeping the query-based method name "allCookies" doesn't work well. It reads weird.

We shouldn't model the repo like a collection in this case. Collections are designed to be queried. Make it something different entirely. Maybe like this:

    #!swift
    let handler: ([Cookie]) -> () = { cookies in /* ... */ }
    let request = CookieFetchRequest(includesIngredient: ["chocolate", "nuts"])
    cookieRepo.execute(request, handler)

`CookieFetchRequest` is understood by the repo. `CookieFetchRequest` may be a special case of a `CollectionFetchRequest<T>` which uses generics to determine which element type the resulting array should contain. This way we can use fetch requests generally in different repositories and override special case implementations.

Executing fetch requests is a common pattern in Core Data. Why not use it in our own code?

The drawback to me is that working with collection-like interfaces is _easy_. It fits my mental model very well. Executing fetch requests is way too technical lingo for my taste. I wouldn't want it to be part of an expressive domain.

Then again, sometimes we don't need expressive domains but mere data containers and data access objects. Maybe we shouldn't call it "repository" then. 

The big advantage of `execute(request:, handler:)` is this: we can do execution asynchronously inside the repository, or whatever we decide to call it. The client can fire off execution of 10 or 100 fetch requests. When you use return values, you [expect immediate results][3ways]. When you pass a handler block, you cannot sensibly expect anything in particular to happen immediately.

Repositories usually return values immediately. If it takes longer to fetch the objects, the thread blocks until fetching is finished. Using the principles of East-Oriented Code, we end up with a non-blocking alternative which apparently doesn't read so well. 

Is this worth the switch away from the pretty well understood mental model of collections?

I have to more experimentation in real-world code to find out what a [functional core](https://www.destroyallsoftware.com/talks/boundaries) might look like and how it'd help.

**What do you think?** How do you make fetching objects asynchronous and readable?

[3ways]: /posts/2015/03/3-ways-model-relationship-domain/
