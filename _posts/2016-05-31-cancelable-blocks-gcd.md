---
title: TIL You Can Cancel Enqueued GCD Blocks
created_at: 2016-05-31 17:00:45 +0200
kind: worklog
tags: [ gcd, closure ]
url: http://www.mattrajca.com/2016/04/23/canceling-blocks-in-gcd-copy.html
comments: on
---

Today I learned that you can cancel a delayed `dispatch_block_t` with the new `dispatch_block_cancel` (available since OS X 10.10/iOS 8.0). Thanks [Matt][matt] for the post -- here's a Swift example:

    #!swift
    let work = dispatch_block_create(0) { print("Hello!") }
    
    # Execute after 10s
    let delayTime = dispatch_time(DISPATCH_TIME_NOW, Int64(10 * Double(NSEC_PER_SEC)))
    dispatch_after(delayTime, dispatch_get_main_queue(), work)
    
    dispatch_block_cancel(work)
    # Will never print "Hello!"

**Note:** canceling doesn't work if the block is being executed.

If I knew that this API existed, I might not have used the very cumbersome approach from below in [Move!](http://christiantietze.de/move-work-break/).


## Super-Weird Legacy Version of a Cancelable Delayed Block

For historic purposes, here's an adaptation of the cancelable dispatch block you may find on the internet that I once have adapted for Swift:

    #!swift
    typealias CancelableDispatchBlock = (cancel: Bool) -> Void

    func dispatch(cancelableBlock block: dispatch_block_t, atDate date: NSDate) -> CancelableDispatchBlock? {
    
        // Use two pointers for the same block handle to make
        // the block reference itself.
        var cancelableBlock: CancelableDispatchBlock? = nil
    
        let delayBlock: CancelableDispatchBlock = { cancel in
        
            if !cancel {
                dispatch_async(dispatch_get_main_queue(), block)
            }
        
            cancelableBlock = nil
        }
    
        cancelableBlock = delayBlock
    
        let interval = Int64(date.timeIntervalSinceNow)
        let delay = interval * Int64(NSEC_PER_SEC)
    
        dispatch_after(dispatch_walltime(nil, delay), dispatch_get_main_queue()) {
        
            guard let cancelableBlock = cancelableBlock else { return }

            cancelableBlock(cancel: false)
        }
    
        return cancelableBlock
    }

    func cancelBlock(block: CancelableDispatchBlock?) {
    
        guard let block = block else { return }
        
        block(cancel: true)
    }

The trick is this: the delayed block `delayBlock: CancelableDispatchBlock` captures its context where a reference to `cancelableBlock` is included -- but not set yet. Then you make the reference point to the `delayBlock` itself.

The actual canceling is a fake, though. The block is still called. It aborts early if the `cancel` parameter is `true`, though.

[matt]: http://www.mattrajca.com/2016/04/23/canceling-blocks-in-gcd-copy.html
