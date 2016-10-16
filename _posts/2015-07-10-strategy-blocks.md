---
title: Using Blocks to Realize the Strategy Pattern
created_at: 2015-07-10 22:24:31 +0200
kind: worklog
tags: [ block, swift, design-pattern ]
vgwort: http://vg08.met.vgwort.de/na/efc1931940d24d3daf227e8a20c0227d
comments: on
---

There's this saying that the [Strategy pattern](https://en.wikipedia.org/wiki/Strategy_pattern) can be realized in Swift using blocks. Without blocks, a Strategy object implements usually one required method of an interface (or protocol) to encapsulate a variation of behavior. This behavior can be switched at runtime. It's like a plug-in.

Well, blocks can do the same. They can become attributes of an object and be switched out. They also capture context if necessary, which may sometimes be a bonus. The only drawback is that blocks can't encapsulate state of their own except the captured context. 

Here's a real-world example from a recent project. It's a work break timer. It deals with two types of timers, realized via dispatch queues: one for work, and one for breaks. The break timer should restart when it's prolonged, the work timer should continue to tick if it's started, or start itself if it wasn't already.

Here's a Strategy-based version of the difference in prolongation behavior:

    #!swift
    protocol TimerProlongationStrategy {
        func prolong(timer: TimerType)
    }

    struct StartOnceTimerProlongationStrategy: TimerProlongationStrategy {
    
        func prolong(timer: TimerType) {
        
            if timer.isActive {
                return
            }
        
            timer.start()
        }
    }

    struct ResetTimerProlongationStrategy: TimerProlongationStrategy {
    
        func prolong(timer: TimerType) {
        
            if timer.isActive {
                timer.prolong()
                return
            }
        
            timer.stop()
            timer.start()
        }
    }

That's very verbose, but it's straightforward to use:

    #!swift
    class TimerCoordinator {
    
        var workTimer: Timer!
        var breakTimer: Timer!
    
        init(workDuration: Minutes, breakDuration: Minutes) {
        
            self.workTimer = Timer(duration: workDuration.seconds, 
                scheduler: self, 
                prolongationStrategy: StartOnceTimerProlongationStrategy(), 
                block: finishWork)
            self.breakTimer = Timer(duration: breakDuration.seconds, 
                scheduler: self, 
                prolongationStrategy: ResetTimerProlongationStrategy(), 
                block: finishBreak)
        }
    }

Instead of setting up two `Timer` types, I can use one type and delegate variation to the `prolongationStrategy` attribute.

With blocks put in place of Strategy objects, it would look like this:

    #!swift
    class TimerCoordinator {
    
        var workTimer: Timer!
        var breakTimer: Timer!
    
        init(workDuration: Minutes, breakDuration: Minutes) {
        
            self.workTimer = Timer(duration: workDuration.seconds, 
                scheduler: self, 
                prolongationStrategy: { timer in
                    if timer.isActive {
                        return
                    }
        
                    timer.start()
                }, 
                block: finishWork)
                
            self.breakTimer = Timer(duration: breakDuration.seconds, 
                scheduler: self, 
                prolongationStrategy: { timer in
                    
                    if timer.isActive {
                        timer.prolong()
                        return
                    }
        
                    timer.stop()
                    timer.start()
                },
                block: finishBreak)
        }
    }

That does read even worse than the version before!

But notice that I've referenced `finishWork` and `finishBreak` respectively as the last argument of the initializer. Instead of a `() -> Void` block, I pass in the reference to a method of `TimerCoordinator`.

Strategies don't have to be realized as in-line blocks or objects. They can be realized as methods or free functions, too.

Using functions (because methods don't make much sense for this use case), the full code will look like this:

    #!swift
    func startOnceTimerProlongationStrategy(timer: Timer) {
    
        if timer.isActive {
            return
        }

        timer.start()
    }
    
    func resetTimerProlongationStrategy(timer: Timer) {
        
        if timer.isActive {
            timer.prolong()
            return
        }

        timer.stop()
        timer.start()
    }
    
    class TimerCoordinator {
    
        var workTimer: Timer!
        var breakTimer: Timer!
    
        init(workDuration: Minutes, breakDuration: Minutes) {
        
            self.workTimer = Timer(duration: workDuration.seconds, 
                scheduler: self, 
                prolongationStrategy: startOnceTimerProlongationStrategy, 
                block: finishWork)
                
            self.breakTimer = Timer(duration: breakDuration.seconds, 
                scheduler: self, 
                prolongationStrategy: resetTimerProlongationStrategy,
                block: finishBreak)
        }
    }

This gets around inline blocks which are hard to read and doesn't introduce unnecessary objects.

Blocks are nice as they are, but functions as first-class citizens of Swift are even nicer because handles to them can be passed instead of blocks.

Using functions for this will work only if you don't need to have stateful Strategy instances. In my case, the Strategy objects were simple wrappers around real functions, so it worked nicely.
