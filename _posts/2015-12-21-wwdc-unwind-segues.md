---
title: "Segues: How to Use Them While Saving Lines of Code"
created_at: 2015-12-21 11:05:56 +0100
kind: worklog
tags: [ segue, storyboard, protocol, pop ]
comments: on
---

In ["What's New in Storyboards" of WWDC 2015](https://developer.apple.com/videos/wwdc/2015/?id=215), we're presented with implementations of unwind segues. Now I've stated part of my concerns with [segues and app architecture][seg] already. The sample code was very straight to the point:

    #!swift
    @IBAction func unwindFromNewJournalEntry(segue: UIStoryboardSegue) { 
        let source = segue.sourceViewController as! NewJournalEnrtyViewController 
        addJournalEnrty(source.journalEntry) 
    }

That's a great example of how to obtain data from segues:

1. You fetch the source or destination view controller, depending on the direction.
2. You cast it to some known type. (That part sucks.)
3. You fetch data from it or set an attribute, again depending on the direction.

This couples both parts without any help from the compiler, though. The associated view controllers could change to anything in your Storyboard and you'll end up with a runtime exception.

Segues don't play well with more explicit architectural patterns. I think that sucks. This is a very very quick shortcut to pass data from A to B or back again. It comes with a cost, though. Better be aware of it. I've got a small app which uses segues extensively and will benefit from a bit of re-thinking and refactoring towards more idiomatic segue code. But I won't use segues everywhere.

There's no way to circumvent casting source or destination view controller to the expected type, although preferably with if-let. There's a better way to start and unwind from segues, though, getting rid of the hard-coded string literals in at least two places.

When I call segues, I will use them with [Natasha's cool `SegueHandlerType` protocol extension:](http://natashatherobot.com/protocol-oriented-segue-identifiers-swift/)

    #!swift
    class ViewController: UIViewController, SegueHandlerType {

        enum SegueIdentifier: String {
            case ProcessOrder
            case CancelOrder
        }
    
        override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
        
            switch segueIdentifierForSegue(segue) {
            case .ProcessOrder: // ...
            case .CancelOrder: // ...
            }
        }
    
        @IBAction func cancel(sender: AnyObject) {

            performSegueWithIdentifier(.CancelOrder, sender: self)
        }

        @IBAction func process(sender: AnyObject) {
            
            performSegueWithIdentifier(.ProcessOrder, sender: self)
        }
    }

The `SegueHandlerType` protocol comes with a protocol extension that takes care of the ennerving `rawValue` enum dance. Here's the protocol definition for completeness:

    #!swift
    import UIKit
    import Foundation
 
    protocol SegueHandlerType {
        typealias SegueIdentifier: RawRepresentable
    }

    extension SegueHandlerType where Self: UIViewController,
        SegueIdentifier.RawValue == String
    {
    
        func performSegueWithIdentifier(segueIdentifier: SegueIdentifier,
            sender: AnyObject?) {
        
            performSegueWithIdentifier(segueIdentifier.rawValue, sender: sender)
        }
    
        func segueIdentifierForSegue(segue: UIStoryboardSegue) -> SegueIdentifier {
        
            guard let identifier = segue.identifier,
                segueIdentifier = SegueIdentifier(rawValue: identifier) else { 
                
                    fatalError("Invalid segue identifier \(segue.identifier).") 
            }
        
            return segueIdentifier
        }
    }

In [her comments](http://natashatherobot.com/protocol-oriented-segue-identifiers-swift/#comment-2415684609), someone pointed out that this makes reading the code harder for the next programmer only. I agree that this protocol does introduce a new concept that has to be understood first -- but it's hardly an issue since _everything_ you add in terms of indirection, for example, has to be read and understood sooner or later. The `SegueHandlerType` encapsulates a lot of segue-handling noise. Once you get that, it's easy as pie to understand `performSegueWithIdentifier(_:, sender:)` or `segueIdentifierForSegue(_:)`.

I worry more about the local coupling of segue definition and segue handling. There's no way to use segues from two different locations in code without subclassing the master segue handler. (But then you cannot add new segues in subclasses.) Will have to look at this in practice some more to find out if this becomes a real pain.

[seg]: /posts/2015/01/segues-vs-tell-dont-ask/
