---
title: Use and Misuse of Helper Functions
created_at: 2016-01-30 13:49:27 +0100
kind: worklog
tags: [ swift, closure ]
comments: on
---

In Swift, you can define functions within functions. These are often referred to as **helpers.** [Rob Napier][rn] brought this to my attention recently. Like any other closure, helper functions capture variables from their surrounding context if needed. This way you don't have to pass them along as parameters.

Here's Rob's example with comments added:

    #!swift
    func solve(config: Config) throws {
        let input = try config.parser(filename: config.filename)

        let formatter = NSDateComponentsFormatter()
        formatter.allowedUnits = [.Hour, .Minute, .Second]

        let startDate = NSDate()

        // Closure captures `formatter` and `startDate`
        func duration() -> NSString {
            return formatter.stringFromDate(startDate, toDate: NSDate()) ?? ""
        }

        var bestSolution: Solution?
        for solution in GeneratorSequence(config.makeSolutionGenerator(input)) {
            bestSolution = solution
            print("\(duration()) : \(solution.progressOutput)")
        }
        print("Final Time: \(duration())")
        print("")
        print(bestSolution?.finalOutput ?? "NO SOLUTION")
    }

If `solve(_:)` was a method in a class, for example, I traditionally would've extracted `duration()` into a private method of the type. Private methods are your traditional helpers to make code readable and compose the actions of the public interface of parts readers can manage to understand.

The thing is that helpers look odd. They are written in place and thus don't really help get an overview of the function you call, which is the helper's context. Since they capture the context, it's not as easy to find out where a referenced variable comes from compared to parameters plainly visible in the signature.

Also, I'm kind of afraid that resorting to helpers makes refactoring less likely. You don't have to create a helper object, for example, to encapsulate an algorithm, because you can organize your code with helpers.

I have never used these in production and I have a hard time finding utility for them. Do you use these? When and why?

[rn]: https://gist.github.com/rnapier/d58e893234539065222a
