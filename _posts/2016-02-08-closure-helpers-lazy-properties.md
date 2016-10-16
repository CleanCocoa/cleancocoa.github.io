---
title: Inline Blocks as Result-Producing Helpers
created_at: 2016-02-08 14:13:57 +0100
kind: worklog
tags: [ closure, swift ]
comments: on
---

In a recent post, I wasn't too fond of [inline helper functions](/posts/2016/01/helper-functions/). 

    #!swift
    func displayComplexComputation() {
        
        // Outer scope
        let x = numberOfBananas()
        
        func subComputation() -> Int {
            let y: Int = ...
            // Access in inner scope through capturing in this closure
            return x * (y + someOtherFactor)
        }
        
        let result = subComputation() + ...
        
        someView.showResult(result)
    }
    
Similar things can be accomplished with blocks instead of functions, as both are closures that capture their contexts. Blocks don't even have to have a name. Inner functions and blocks share the same semantics. Found in [Little Bites of Cocoa #180](https://littlebitesofcocoa.com/180-more-swift-tricks):

    #!swift
    enum Theme {
      case Day, Night, Dusk, Dawn
    
      func apply() {
        // ...
    
        let backgroundColor: UIColor = {
          switch self {
            case Day: return UIColor.whiteColor()
            case Night: return UIColor.darkGrayColor()
          }
        }()
    
        // ... set backgroundColor on all UI elements
      }
    }


I love how lazy properties are handled in Swift. This is exactly the same approach. Bundling a sequence of set-up instructions into a closure helps readability, I find.

Take for example constructing and formatting a `NSDate` from `NSDateComponents`, which is always taking at least 4 lines of code:

    #!swift
    let timeFormatter: NSDateFormatter = ...
    let hour: UInt = ...
    
    let timeOfDay: String = {
        let comps = NSDateComponents()
        comps.hour = hour
    
        let calendar = NSCalendar.currentCalendar)
        let date = calendar.dateFromComponents(comps)
        
        return timeFormatter.stringFromDate(date)
    }()

You can use this for setup of complex service objects, too, of course.

    #!swift
    showWindowService = {
        let aggregation = Aggregation(appRepository: appRepository, dataSource: dataSource)
        let interactor = Interactor(aggregation: aggregation)
        let analyticsWindowController = AnalyticsWindowController()
        let presenter = Presenter(interactor: interactor, view: analyticsWindowController)

        return ShowAnalyticsWindow(interactor: interactor, presenter: presenter)
    }()
    
