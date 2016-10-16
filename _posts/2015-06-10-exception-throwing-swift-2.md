---
title: "Be Cautious with Try-Catch for error handling in Swift 2.0"
created_at: 2015-06-10 18:26:41 +0200
kind: worklog
tags: [ swift, control-flow, exception ]
shared_image: "blog/201503012144_guard-t.jpg"
vgwort: http://vg08.met.vgwort.de/na/e696acd193b3444aae4d4ce02e5717da
comments: on
---


Throwing exceptions and being able to catch them is awesome news. Then there are cases which can be handled with try-catch technically, but shouldn't.

Think about this: in what cases could you make use of that? And what are the consequences for your work?

For example, Erica Sadun [points out][sadun] that accessing array indexes out of bounds as a subscript will still crash the app since you can't catch that kind of exception (yet).

There's two things you should be cautious of:

* **You might be tempted to be more sloppily with arrays.** "If the index is out of bounds, who cares, let's just recover from that condition." -- This will promote other bad decisions (slippery slope!). Also, you might not be as inclined to expose custom accessors in your objects to their collection properties because error prevention dropped down a few points your priorities list.
* **You might be tempted to use the catch blocks for different paths of program flow.** Go ahead and blindly access an array; surround with do-try-catch. If it didn't work, tell the user to enter some data. If it works, display data. -- Now the language feature is not used to handle a real _exception_ but rather as a replacement for other conditionals, like `if`, to control the program flow. That's not good at least because the `throws` clause clutters your code and marks functions as potentially unsafe without any need.

It's always preferable to not throw exceptions whenever you can. Try to recover from non-optimal situations by other means.

There's a dangerous path ahead: if you throw an exception anytime the program doesn't execute in the ideal way -- if it doesn't follow the happy path --, then you will think less about preventing non-happy circumstances. You will program less defensive. Just toss in a catch-all.

With Swift 1.x, we had to establish new patterns and adopt practices from functional languages. Also, [error callback blocks][callb] we got to know from Objective-C still worked as expected (and still do).

That said, recently I found it vexing recently to solely rely on the [Result enum][result] for the purpose of parsing plist data. I could've done with optionals, but then it'd be hard to pin down the cause of the problem. Throwing a String description of the problem during parsing would have been just as nice: quickly escape from the nested object dependencies to report a problem.

It turned out the Result enum worked just fine, though. I was able to nest potentially failing operations in blocks using `flatMap` on the result:

    #!swift
    public func exerciseData() -> Result<ExerciseData, String> {
        
        return exerciseCatalogue().flatMap { catalogue in
            tierHierarchy().flatMap { tiers in
                return .success(ExerciseData(tierHierarchy: tiers, 
                    exerciseCatalogue: catalogue))
            }
        }
    }
    
    private func exerciseCatalogue() -> Result<ExerciseCatalogue, String> { /* ... */ }
    private func tierHierarchy() -> Result<TierHierarchy, String> { /* ... */ }

The Result enum helps to guard against failure. Optionals would kind of do the same in this case.

There's [another way][owensd] to work with the Result enum nowadays since `guard` was introduced and all:

    #!swift
    func base() -> Result<(), String> { /* ... */ }
    
    func handle() {
        let result = base()

        guard let value = result.value else {
            print("Error: \(result.error!)")
		
    		return
        }
    
        print("value: \(value)")
    }

I like that `guard` works just like `if` with clearer intent, only here we propagate the optional success value to the scope of `handle()` instead of keeping it inside the conditional's. Finally, no more nested conditions for the happy path! Decreasing the level of indentation in your methods is always good, but Swift made this really hard until now.

Then again I'd be happy to just assume a non-optional variable and raise exceptions when initializing it from plist data.

The point of the parser is to read a graph of data and verify data integrity. The app will need to crash if the data it uses is corrupt. That's the point of all this. And I'm going to use it as a means of validation before shipping the app. If I want to abort app execution anyway during development only, I might just as well throw an exception and never catch it.

What would you use the new do-try-catch for? Where'd you stick to established patterns?

[sadun]: http://ericasadun.com/2015/06/09/swift-why-try-and-catch-dont-work-the-way-you-expect/
[result]: /posts/2015/05/nicest-result-enum-ever/
[callb]: /posts/2015/04/error-callbacks-objective-c/
[owensd]: http://owensd.io/2015/06/09/swift-v2-error-handling-revisit.html

