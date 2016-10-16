---
title: Jiggle GCD Queues to Find Problems
created_at: 2015-11-13 16:57:20 +0100
kind: worklog
tags: [ testing, gcd, concurrency, asynchrony ]
comments: on
preview: fulltext
---

To debug my threading issues and help bring forth future problems, I have created a simmple object that slows the current queue down:

    #!swift
    let IsRunningTests = NSClassFromString("XCTestCase") != nil

    struct QueueJigglePoint {
    
        /// Randomly interfere with the thread.
        static func jiggle() {
        
            guard !IsRunningTests else { return }
        
            #if DEBUG
                usleep(2*1000000) // 2 seconds
            #endif
        }
    
    }

I got this idea from Brett Schuchert on pages 188--90 of Uncle Bob's  _[Clean Code. A Handbook of Agile Software Craftsmanship](http://amzn.to/1WWz7rt)_. There, interference with traditional threading should randomly sleep, yield, or fall through. Enqueued blocks are a lot less volatile, so I only came up with sleeping. 

Randomizing the sleep interval is up next. But a fixed number of 2--10 seconds helps find UI-blocking code already.

Just throw in a `QueueJigglePoint.jiggle()` in `NSManagedObjectContext.performBlock` executions, when dispatching async to the background, or when reading files, for example.
