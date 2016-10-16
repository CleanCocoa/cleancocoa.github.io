---
title: Going Beyond Guard Clauses in Swift
created_at: 2015-03-01 20:54:02 +0100
kind: worklog
tags: [ functional-programming, swift, control-flow, design-pattern ]
vgwort: http://vg08.met.vgwort.de/na/6499b8ee0deb41cf956f20056f0bc1d2
comments: on
shared_image: "blog/201503012144_guard-t.jpg"
code: swift
---

Erica Sadun brought up a topic which is bugging me for some time already. It's how Swift encourages you to handle optionals. It's related to [handling exceptions](/posts/2015/02/functional-swift-exceptions/), which I covered earlier.

Either you use `if-let` to unwrap them implicitly and do all the stuff inside the nested code block, or you end up with code which is a lot wordier than I'd like it to be.

I propose all of us try radically different approaches in our projects to find out whether Swift actually encourages something totally different. You find my proposal at the bottom.

First, let's get everyone on the same track.

## Happy Paths and Guard Clauses in Swift

With Swift's `if-let` pattern, the happy path of execution is usually inside the nested block of code. The path of error may extend to the very last line of a method.

It may look like this:

    #!swift
    func workOnSomething(comingFrom aDataSource: Source) {
        if let theThing = aDataSource.thingFromSomewhere() {
            self.worker.work(theThing)
            // this is the happy path; but it's finished, 
            // so don't go on afterwards
            return
        }
        
        // Handle that aDataSource didn't produce theThing, i.e. show
        // an error or otherwise recover from a bad state.
    }

The last line of the method won't tell you what it does. You actually have to start at the beginning and follow conditions in your mind to find out where you'll end up.

In Objective-C, you would've done it thus (assuming it's a method):

    #!objc
    - (void)workOnSomethingComingFrom:(Source *)aDataSource {
        Thing *theThing = [aDataSource thingFromSomewhere];
        
        if (!theThing) {
            // This is the Guard Clause.
            // Handle that aDataSource didn't produce theThing, i.e. show
            // an error or otherwise recover from a bad state.
            return
        }
        
        // This is the happy path, ending naturally.
        [self.worker workOnThing:theThing];
    }

If most of your methods are written in this fashion, it's much more readable. If you want to find out what the happy path is, just ignore all conditions. If you want to know what may go wrong, look at the [Guard Clauses][gc].

It's possible to do it this way in Swift, but then you have to do without implicitly unwrapped optionals, i.e. unwrap them manually:

    #!swift
    func workOnSomething(comingFrom aDataSource: Source) {
        let maybeAThing = aDataSource.thingFromSomewhere()
        if maybeAThing == nil {
            // This is the Guard Clause.
            // Handle that aDataSource didn't produce theThing, i.e. show
            // an error or otherwise recover from a bad state.
            return
        }
        let theThing = maybeAThing!
        
        // This is the happy path, ending naturally.
        self.worker.work(theThing)
    }


Erica called this linearizing the code. You get rid of nesting stuff. If you have to set a `NSErrorPointer`, the Guard Clauses will become even more involved. Look at Erica's example:

    #!swift
    func StringFromURL(url : NSURL, error errorPointer : NSErrorPointer) -> (NSString?) {
        var error : NSError?
    
        // Fetch data. Blocking.
        let data_ = NSData(contentsOfURL:url, options: NSDataReadingOptions(rawValue:0), error: &error)
        if (data_ == nil) {
            if errorPointer != nil {
                errorPointer.memory = error
            }
            return nil
        }
        let data = data_!
    
        // Convert to string.
        let string_ = NSString(data: data, encoding: NSUTF8StringEncoding)
        if (string_ == nil) {
            if errorPointer != nil {
                errorPointer.memory = BuildError(1, "Unable to build string from data")
            }
            return nil
        }
        let string = string_!
    
        return string
    }

[Guard clauses][gc] are used heavily. Compared to Objective-C, the Swift version is "wordier", as Erica says. I agree. You need two nested `if` conditions for each of the two steps, and you need to manually unwrap the optional.

This is by design: you should only force unwrap conditionals if you are 100% positive it'll work, and the bang will remind you what's going on.

But nobody likes to add a bang all the time. I think this can be streamlined.

[gc]: http://c2.com/cgi/wiki?GuardClause

## Delegate Guard Clauses to Helper Functions

The pattern works as follows:

* obtain data and put it into a variable,
* check data for erroneous condition,
* in the case of error: abort (`return nil`), and set an `NSError` if needed; or
* in the happy case: go on with the flow (and eventually return the variable).

Erica's example uses this twice. Once to obtain data from a URL, once to convert the data to a string.

Can't we make this easier to write?

Programming Ruby taught me a lot. I favor super-focused methods. Let's refactor Erica's example code first to slim `StringFromURL` down.

There are two Guard Clauses. This indicates two potentially failing processes are going on here. One for obtaining data, one for transforming data.  `StringFromURL(_:,_:)` is just the higher-order wrapper of these two operations.

    #!swift
    func StringFromURL(url : NSURL, error errorPointer : NSErrorPointer) -> (NSString?) {
        let data = fetchData(url, error: errorPointer)
        if (data == nil) {
            return nil
        }
    
        let string = stringFromData(data!, error: errorPointer)
        if (string == nil) {
            return nil
        }
        
        return string
    }
    
    func fetchData(url : NSURL, error errorPointer : NSErrorPointer) -> NSData? {
        var error : NSError?
    
        // Fetch data. Blocking.
        let data = NSData(contentsOfURL:url, options: NSDataReadingOptions(rawValue:0), error: &error)
        if (data == nil) {
            if errorPointer != nil {
                errorPointer.memory = error
            }
            return nil
        }
        
        return data
    }
    
    func stringFromData(data : NSData, error errorPointer : NSErrorPointer) -> NSString? {
        let string = NSString(data: data, encoding: NSUTF8StringEncoding)
        
        if (string == nil) {
            if errorPointer != nil {
                errorPointer.memory = BuildError(1, "Unable to build string from data")
            }
            return nil
        }
        
        return string
    }

Both `fetchData` and `stringFromData` look like before. There's no need to distinguish `data_: NSData?` from `data: NSData!` in any place, which is a win in my opinion. Both look similar and are a little burden when reading the code.

Also, the higher-order function `StringFromURL` doesn't need to worry about the optional `errorPointer` because the helper functions take care of this. `StringFromURL` delegates all of the error handling to the other functions.

    #!swift
    func StringFromURL(url : NSURL, error errorPointer : NSErrorPointer) -> (NSString?) {
        fetchData(url, error: errorPointer).map { data in
            stringFromData(data, error: errorPointer) {
                
            }
        }
        if (data == nil) {
            return nil
        }
    
        let string = stringFromData(data!, error: errorPointer)
        if (string == nil) {
            return nil
        }
        
        return string
    }

## Striving to be concise: Monadic Approach

I wanted to make something even more concise. Instead of optimizing the procedural approach, I tried to emulate what the [shell pipeline][pipe] does: chain transformation together.

In pseudo-pseudo-code:

    maybeString = fetchData from URL | string from data | string from NSString

It's called `maybeString` because the operation could fail.

Usually, we'd still end up passing an `NSErrorPointer` around. But as [Javier Soto][soto] and others point out: the error pointer and the return value are not conceptually coupled: you can return nil without setting the error pointer, and you can set the error pointer without returning nil.

So two of four possible combinations of return value and error pointer are bogus. It's really meant to be only two options.

You can use an `enum` instead to provide either a happy-path-value or an `NSError`-value:

    #!swift
    enum Result<T> {
        case Value(Box<T>) 
        //         ^^^ Box<T> instead of just T needed in Swift 1.1. to compile
        case Error(NSError)
    }

The client call will look like the following:

    #!swift
    let zenAPI = NSURL(string: "https://api.github.com/zen")
    let stringResult: Result<String> = StringFromURL(zenAPI!)

    switch (stringResult) {
    case .Value(let value):
        // work with value.unbox (of type String)
    case .Error(let error):
        // handle error case
    }

The resulting `StringFromURL` method now is implemented as follows:

    #!swift
    func StringFromURL(url : NSURL) -> Result<String> {
        return fetchData(url).transform { stringFromData($0) }.transform { stringFromNSString($0) }
    }

There you see the pipe-like transformations. (`transform(_:)` is actually `flatMap(_:)` in functional programming terms.)

Look at [Javier's blog post][soto] for details on the `Result<T>` implementation.

I have put [a working example into a Gist][gist] online for your convenience. There, both the conventional and procedural `StringFromURL`, and the alternate version, called `StringFromURLResult`, exist next to each other. If you get rid of the conventional version, you can of course shorten the file a lot and remove `fetchData` in favor of its sibling `fetchDataResult`.

[soto]: http://www.javiersoto.me/post/106875422394
[pipe]: http://en.wikipedia.org/wiki/Pipeline_%28Unix%29
[gist]: https://gist.github.com/DivineDominion/21446ef37ac87c62567b

## Conclusion: This May Be Useful

Overall, I think the `Result` return type can be put to good use in cases like this: obtaining optional data, transforming optional data.

`stringFromNSString(_:)` is actually a really bad example. There's no error case. It's just there for the sake of chaining transformations.

I still have to find out when the overhead of the more functional approach is warranted. Including the `Result` enum once and using it in various places reduces perceived overhead, of course.

But then you buy into `switch-case` on the client side instead of `if-let`. Does this make your code better? Easier to read? Easier to extend?

I argue it's *Yes!* for extension. Adding yet another transformation is easy and independent from the rest of your code. Transformations are pluggable.

I'm not so sure about the other trade-offs. I still have to use this in production.

* **What do you think of this approach?** 
* When would you use it?
* Why would you not use this stuff at all and hold on to the old Cocoa way?

---

Photo by Fagata, License: <a href="http://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0</a>, via <a href="http://commons.wikimedia.org/wiki/File:Stra%C5%BCnik_przed_Pa%C5%82acem_Buckingham;_Buckingham_guard.JPG">Wikimedia Commons</a>
