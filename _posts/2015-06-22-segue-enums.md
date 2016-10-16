---
title: Using Swift Enums to deal with Storyboard Segues
created_at: 2015-06-22 15:59:37 +0200
kind: worklog
tags: [ swift, segue, enum ]
comments: on
---

I like when the code is explicit. Unlike [Brent Simmons](http://inessential.com/2014/07/14/string_constants), I don't feel comfortable using string literals in code especially not when I'm still fleshing things out and change values in Interface Builder a lot.

In Objective-C, I would've used plain old constants. For storyboard segues, for example:

    #!objc
    NSString const * kSegueToThere = @"goThere";
    NSString const * kUnwindSegueFromThere = @"unwindFromThere";

I usually ended up writing methods like `isUnwindSegueForXYZ:` to get rid of the repeating string equality check noise. That's a lot more boilerplate code. Both don't read well.

    #!objc
    - (void)prepareForSegue:(UIStoryboardSegue *)segue {
    
        if ([segue.identifier isEqualToString:kSegueToThere]) {
            // ...
        } else if ([self isUnwindSegueFromThere:segue]) {
            // ...
        }
    }

Brent has a point, though: it doesn't improve readability a lot, especially since I adopt naming conventions which are telling the intent: uppercase identifiers designate a view controller scene; lowercase identifiers designate a segue; lowercase identifiers starting with "unwind", well, guess what.

With Swift, I now have a lot more readable way to group constants: enums.

    #!swift
    enum Destination: String {
        case There = "showThis"
        case Elsewhere = "showThat"
        
        init?(_ segue: UIStoryboardSegue) {
            self.init(rawValue: segue.identifier!)
        }
    }

In `prepareForSegue(_:,sender:)`, where these values are used the most, I end up with this:
    
    #!swift
    override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        
        switch Destination(segue)! {
        case .There: // ...
        case .Elsewhere: // ...
        }
    }

I'm very happy with that.

I feel comfortable force-unwrapping the enum in some cases. It's okay when there should be no unknown case. If that's not your favorite way to create runtime errors, using the new `guard` statement in Swift 2.0 will produce a much more lenient alternative.

**Update 2016-04-25**: Protocol extensions as mixins to the rescue, you can make things even better [with enums + associated values.](/posts/2016/04/type-safe-segues/)
