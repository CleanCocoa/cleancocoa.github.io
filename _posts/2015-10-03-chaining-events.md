---
title: Chaining Events, Take One
created_at: 2015-10-03 14:11:22 +0200
kind: worklog
tags: [ event-sourcing, domain-event, ddd ]
comments: on
---

In an execellent book [on Domain-Driven Design][dddbk] that I'm reading in my spare time, they make a lot of use of domain events. I've written about [a strongly typed publish--subscribe infrastructure][publ] in March already to get you started. Some of the book samples include what seems to be ad-hoc event subscribers. I try to port that to Swift.

Here's a C# example from the book, page 679:

    #!csharp
    public void Execute(ModifyCategoryCommand command)
    {
        var category = _catalogueRepository.FindBy(command.Id);
    
        using (DomainEvents.Register<CategoryUpdated>(onCategoryUpdated)) 
        {
            category.Update(command);
        }
    }
    
    private void onCategoryUpdated(CategoryUpdated @event)
    {
        _catalogueViewModel.Update(event)
    }

That's registering `onCategoryUpdated` as event handler for a `CategoryUpdated` event being fired by `category.Update(command)`. It reads as if this is a complicated way to pass the result of the `Update` call to `onCategoryUpdated`. 

If events are passed asynchronously, how's the event listener kept alive? When is it de-registered again? Or is the owning object kept alive very long, the handler registered only once, and subsequent executions fall through? I assume the latter, but that doesn't make much sense. If an event is fired but reaches `onCategoryUpdated`, say, 10 minutes late, how's that resulting in a valid view model update? Does the view model discard events that don't match the ID, for example?

I want to achieve single-response listeners first. When it receives an event, the listener can be discarded.

This `using` clause was weird at first. Here's the requirements to achieve that in Swift:

* `DomainEvents.register<T>(block: (T) -> Void)` -- register for events of type `T` and handle them using the block.
* `using(eventListener: NSObjectProtocol, block: () -> Void)` can replicate the code above; `NSObjectProtocol` is what you get when you use `NSNotificationCenter.addObserver`.
* Remove the notification observer (or event listener) after the `using` part for one-off triggers, or keep it alive .

Since there's no built-in `using` command in Swift, here's the problem: after the attacked closure is executed, the surrounding `execute` method would finish as well. There's nothing left to do. The event listener would be dangling around. If we removed it from `NSNotificationCenter` before leaving `execute`, say with the new `defer` command, then the execution of this method might still end before the event is dispatched on who knows which queue.

So our Swift version of `using` has to be asynchronous.

Here's the best idea I had: create an event listener just like above, but automatically remove it after, say, 2 minutes. Maybe even earlier. Essentially, give it a timeout after which it's going to be removed from the `NSNotificationCenter` and deallocated.

This means we need a small helper object which holds on to an event listener reference and a timeout block.

I played around with the names a bit and came up with [a solution][gist] that can be used like this:

    #!swift
    func execute(ModifyCategoryCommand command) {
        
        let category = catalogueRepository.findBy(command.categoryId)
        
        let eventExpectation = subscribeUsing(onCategoryUpdated)

        do {
            try eventExpectation.fulfilExecuting() {
                category.update(command)
            }
        } catch {
            NSLog("Could not update category")
        }
    }
    
    private func onCategoryUpdated(CategoryUpdated event) {
    
        catalogueViewModel.update(event)
    }
    
I made it so `fulfilExecuting` takes block of `() throws -> Void)` and rethrows the error to the caller, that is `execute(_:)`. I found this useful because the Domain changes which are supposed to fire events sometimes can fail. That's unlikely for model changes like `category.update(_:)`. But it will happen with Domain Services which encapsulate a complex sequence. 

Another approach is to not throw (since that's just another kind of `return`) but publish a failure event instead.

You can see most of a working example [as a Gist on GitHub.][gist]

It can handle more complex setups, which don't read as nice, though:

    #!swift
    let eventExpectation = subscribeUsing(onCategoryUpdated)
        .passingTest({ $0.categoryId == aCategoryId }) // do not apply to all events of that kind
        .onFailure({ NSLog("Expected XYZ event but timed out.") })

    try eventExpectation.fulfilExecuting() {
    
        // ...
    }
    
Feedback very welcome!

[gist]: https://gist.github.com/DivineDominion/8c78f1cdd327f4fd87f0
[publ]: /posts/2015/03/event-publisher/
[dddbk]: http://amzn.to/1L4pal2
