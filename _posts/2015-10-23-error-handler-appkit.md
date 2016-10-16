---
title: How I Solve the Unexpected Error Handling User Experience Problem
created_at: 2015-10-23 08:28:09 +0200
kind: worklog
tags: [ error-handling ]
image: screenshot.png
comments: on
---

For the upcoming Word Counter upgrade, I developed a new feature module in isolation. The outcome is very interesting, as I am including it as a Swift module, that is as a real framework, into the existing app. I need some more time to write about the resulting architecture, but I found the module boundary to be really helpful. Today, though, I want to share my [`ErrorHandling` code][github].

{% include figure.md src="/assets/blog/2015/screenshot.png" alt="Screenshot of error handler" caption="ErrorAlert showing an error" %}

In the course of developing the module I decided to not defer handling errors like failing `NSManagedObjectContext` saves until the very last moment. Instead, I try to gracefully recover from bad state, keep [transactions][trans] short and focused, and report weird stuff to the developer (me) as soon as possible.

So this is what my code looks like now:

    #!swift
    guard let file = fileStore.fileWithID(fileID) else {
        applicationError("Could not detach file \(fileID) from group because it wasn't found.")
        return
    }

    unitOfWork()
        .onError(reportUnitOfWorkError("Could not detach file \(fileID) from its group"))
        .execute({ file.detachFromGroup() })

Wheras `reportUnitOfWorkError` is simply dispatching an error back to the main thread; since a `UnitOfWork` executes blocks on background threads, this is a necessary evil. I tried to make this as nice as possible, making this function curried:

    #!swift
    func reportUnitOfWorkError(message: String = "Unexpected error executing unit of work", 
        function: String = __FUNCTION__, 
        file: String = __FILE__, 
        line: Int = __LINE__)(error: UnitOfWorkError) {
    
        dispatch_async(dispatch_get_main_queue()) {
            applicationError("\(message): \(error)", function: function, file: file, line: line)
        }
    }

You use it like so:

    #!swift
    reportUnitOfWorkError("Something did go wrong!")(error)

That, on its own, would be pretty weird. But you can omit the second function call and obtain something equivalent to a block which you can pass around, just like your old function pointers.

    #!swift
    let customizedErrorReport = reportUnitOfWorkError("OH NO SOMETHING BROKE!")
    let error: NSError = ...
    customizedErrorReport(error)

And in the end, an alert is presented, prompting the user to send a report back to me.

I uploaded all the code [to GitHub][github] for you to check out and build upon.

It's good to get started with defensive programming early. Think of this as an investment in future error reporting mechanisms which may be more elaborate if you like.

[github]: https://github.com/DivineDominion/ErrorHandling
[trans]: /posts/2015/10/unit-of-work-core-data-transaction/
