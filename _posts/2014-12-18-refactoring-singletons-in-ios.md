---
title: Making Good Use of Singletons in Refactoring the iOS App Calendar Paste
created_at: 2014-12-18 13:38:27 +0100
kind: worklog
tags: [ calendarpasteapp, refactoring, programming, viper, singleton ]
vgwort: 964051865a884739827a4255407aab05
comments: on
---

Like I promised last weekend, I am going to write about the process of cleaning up the already rotten source code of _Calendar Paste_. In order to break massive view controllers into manageable pieces and [un-tangle everything][explore], I have to make sure that I don't break the current implementation. Calendar Paste didn't have any automated tests in place. To change this fact is my first priority.

Thanks to Michael Feathers' _[Working effectively with legacy code][leg]_,[^aff] I learned a few techniques to make entangled legacy code testable. Michael's book is full of clever tips, and I highly recommend taking a good look at it.

So here's my confession: For the last two years, I have performed manual tests only. I once knew which change could break what, and so I clicked my way around in the live app to perform manual regression testing.

Manual testing, of course, doesn't scale well during [refactoring][] when you need to ensure that your changes are side-effect free.

First, I am going to create objects to perform repeatedly executed tasks. Calendar Paste has a single user preference at the moment: it stores the default calendar. There are three places in the app where the default calendar is used. All three places access the preferences store, `NSUserDefaults`, directly. I didn't want to use global variables back then, so I thought using a shared data store was a good idea.

`NSUserDefaults`, used in this way, are no different from global variables except maybe that you don't have to worry about concurrency issues. There's another mechanism to encapsulate global state: using your own [singleton objects][singleton].

Using your own singleton objects enables you to use test doubles to verify the calls. The key to make calls to common Cocoa service objects like `NSUserDefaults` and `NSNotificationCenter` testable is to not obtain your instances via the framework's standard means in every place. Wrap the vending of their instances in your own service class. Test doubles are necessary to verify outgoing calls to collaborating objects, and you can provide these for your own service vendors pretty easily.

Testing for `NSUserDefaults` calls is hard if you use its `sharedUserDefaults` singleton accessor everywhere. You could hold a strong reference to the instance and pass it around throughout the app, or you can create a wrapper like I do.

So now I'm going to extract global state into singleton objects before anything else and show you the details to make your code so much easier to test.

[singleton]: https://en.wikipedia.org/wiki/Singleton_pattern
[refactoring]: http://en.wikipedia.org/wiki/Code_refactoring
[leg]: http://www.amazon.com/gp/product/0131177052/ref=as_li_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0131177052&linkCode=as2&tag=chritietwork-20&linkId=SKJQXZUDNCZZNS2T

[^aff]: Affiliate link; I get a small kickback from amazon if you buy from my link.


## Breaking Out Functional Pieces 

Where to start? Well, `AppDelegate` does a lot. Compare its implementation [before](https://github.com/DivineDominion/CalendarPaste-app/blob/01a3a86d40e4064612a7ba256f9f4192f0e49b49/ShiftCal/AppDelegate.m) my refactoring to how it looks [after](https://github.com/DivineDominion/CalendarPaste-app/blob/4c9a25761cac207a33bd8b85271a56f208f445f7/ShiftCal/AppDelegate.m) (or just [see the changes](https://github.com/DivineDominion/CalendarPaste-app/compare/01a3a86d40e4064612a7ba256f9f4192f0e49b49...4c9a25761cac207a33bd8b85271a56f208f445f7#diff-6)).

Before, `AppDelegate` requested access to user's calendar up front, prepared the look and feel of the app, and showed view controllers appropriate to the calendar authorization status. 

Clearly, `AppDelegate` does far more than respond to the frameworks application events. Usually, the application delegate does a lot of stuff for setting up the app. Often, it is even the responder to global events, sent down the responder chain just because you wouldn't know how to handle the event by any other means.

Testing a lot of different concerns is hard. I like it when I don't need to actually test the `AppDelegate` at all. To make it so, the implementation has to be trivially simple, so I extract concerns into their own classes. I don't need to unit test a wrapper if all it does is delegating the message to another object.

In the case of Calendar Paste's `AppDelegate`, I end up with quite a few collaborators:

* **Extract the calendar access guard.** Aptly called `CalendarAccessGuard`, this object shows a "locked" screen until the user grants calendar access to the app. I put the calendar authorization request into the single public method of this class and do the rest via delegate callbacks. It's API is still a bit weird and I'm not very happy with it initializing the view controllers, but it's a start.
* **Replace Apple's global singletons with my own.** `CTKNotificationCenter` singleton provides `NSNotificationCenter` objects, only I can replace the return value in tests and not post actual notifications anywhere. `CTKUserDefaults` does the same for `NSUserDefaults`. This way I ensure that every test runs independently. Using the default `NSNotificationCenter` in tests produced weird behavior in my Word Counter test suite.
* **Provide even more singletons.** Add a `UserCalendarProvider` singleton to hold on to the `EKEventStore` needed for writing events and reading user calendars. Primarily, this service object is an adapter for the user's default calendar choice stored in and read from `NSUserDefaults`. With this in place, I can provide test stubs. Granted, singletons aren't the best solution to every problem. They will work for the limited scope of Calendar Paste for a while. I expect to perform a refactoring in the future where I get rid of singletons in favor of [injected dependencies][inj].
* **Isolate Core Data.** I had a so-called template controller in place to create the actual event templates already. From this, I extracted [a `PersistentStack` object][perstack] and now inject an `NSManagedObjectContext` into the template controller instead of creating it there. I also add an `XCTestCase` subclass called `CoreDataTestCase` in which I set up an in-memory context to inject during tests.

These very first steps have one thing in common: **create more seams.**

One single big object is hard to test. The bigger an object, the more internal state it is going to have. Methods tend to become less and less free of side-effects, so your tests have to test more state with every call.

In comparison, the communication between many objects can be handled with ease: if message _X_ comes in, delegate to object _Y_. That's what the application delegate is going to do in the end.

I want to end up with a [VIPER-like architecture](http://www.objc.io/issue-13/viper.html). The `CalendarAccessGuard` will eventually work very differently to accomplish this.

If you take a look at the code, note the absence of a **Application Services** to mediate between the all-combining **Infrastructure** and the **User Interface**. All you'll see is `LockApp` and `UnlockApp`, which are called by the `CalendarAccessGuard` upon its initialization.

There's no real **Domain**, either. Since the app is mostly about managing data, maybe there won't ever be much to be called "Domain". There's a lot of Application Service logic hidden in view controllers, though. To tackle the view controllers, Infrastructure has to be rock solid first. By replacing calls to `NSUserDefaults` in view controllers with my own service object, I took a step into the right direction already.

Let me show you how I did it.

[inj]: http://en.wikipedia.org/wiki/Dependency_injection#Constructor_injection
[perstack]: http://www.objc.io/issue-4/full-core-data-application.html

### Test Singleton Overrides

The 'extract singleton' pattern I applied was repeated multiple times and is used in tests like this:

    @implementation ATestCase
    - (void)setUp {
        [super setUp];
    
        testReplacement = [[SomeTestCollaborator alloc] init];
        TheSingleton *singletonOverride = [TheSingleton singletonWithCollaborator:testReplacement];
        [TheSingleton setSharedInstance:singletonOverride];
        // ...
    }

    - (void)tearDown {
        [TheSingleton resetSharedInstance];
        [super tearDown];
    }
    // ...
    @end

A call to `+setSharedInstance:` overrides the global value, while `+resetSharedInstance` removes it again. Subsequent calls will initialize the default singleton instance.

This way, I can ensure each test case uses its own `NSUserDefaults` replacement, or gets its own `NSNotificationCenter`. Especially shared `NSNotificationCenter` instances have bitten me during tests in the past: only sometimes notifications were passed between test cases and made some tests fail. But if it isn't predictable, it's may just as well become a bug. 

Singleton replacements are possible thanks to `dispatch_once`. Have a look at part of the implementation of `UserCalendarProvider`, for example:

    @implementation UserCalendarProvider
    static UserCalendarProvider *_sharedInstance = nil;
    static dispatch_once_t once_token = 0;

    + (instancetype)sharedInstance {
        
        dispatch_once(&once_token, ^{
            if (_sharedInstance == nil)
            {
                EKEventStore *eventStore = [[EKEventStore alloc] init];
                EventStoreWrapper *wrapper = [[EventStoreWrapper alloc] initWithEventStore:eventStore];
                _sharedInstance = [[UserCalendarProvider alloc] initWithEventStoreWrapper:wrapper];
            }
        });
    
        return _sharedInstance;
    }

    + (void)setSharedInstance:(UserCalendarProvider *)instance {
        
        once_token = 0; // resets the once_token so dispatch_once will run again
        _sharedInstance = instance;
    }

    + (void)resetSharedInstance {
        
        [self setSharedInstance:nil];
    }
    // ...
    @end

Of course, `UserCalendarProvider` provides convenient accessors to expose data of its own collaborators to the outside. You shouldn't be deeply calling methods of an object's property objects but expose accessors for their state in the managing object instead. Instead of calling `[aUserCalendarProvider.eventWrapper.eventStore defaultCalendar]`, for example, call `[aUserCalendarProvider defaultCalendar]`. Makes for a much more readable and easier to change API.

Working with singletons in this way can be a quite powerful enhancement over getting global state by means of `NSUserDefaults`. Put calls to `NSUserDefaults` in a wrapper object with such method names to communicate your intent, and there you go: readable code, fully capable of using test replacements instead of the real Cocoa stuff.


## The Collaborators I End Up With

Whenever I came back to the code of Calendar Paste to fix a bug or change something in the past, I wondered how there could be so little files in the project. This was a clear sign of dealing with too many concerns in one place. But it worked, because I used to remember how to do things.

After this first round of rather innocent refactorings, I end up with a handful new components and drastically improved object interfaces. There are unit tests in place, and seeing how they pass gives me joy.

In the past, some view controllers would've sent queries to `UIApplication.delegate` to obtain the shared `EKEventStore` instance which the application delegate used to hold on to. Such implicit interface contracts won't be easily caught by the compiler. Now I provide convenient access through a designated singleton instead -- and maybe even get rid of that, too, when the time has come.

* `CalendarAccessGuard` ensures the app doesn't leave its initial state until the user authorized calendar access.
* `EventStoreWrapper` provides convenient methods to request authorization and fetch the user's default calendar's identifier. Bonus: instead of testing calls to an `EKEventStore`, I can test calls to `EventStoreWrapper` methods. Remember not to stub code which doesn't belong to you so you don't introduce false assumptions into your code!
* `UserCalendarProvider` is the singleton to ask for the user's default calendar's identifier, and the corresponding `EKCalendar`. Uses `EventStoreWrapper` for this.
* My favorite [TestingToolkit][ttk] wrappers, `CTKNotificationCenter` and `CTKUserDefaults` are used heavily, too.

Refactoring is an incremental process. From the outside in, I strive to make the code more readable, split it into manageable components, and finally be able to add features to the app again without too much worry.

The downside? There's going to be a few more days worts of full-time coding spent without much visible benefit to the user.

Put differently, it's a lot of hours without any increase in revenue. This cleaning up is going to be an investment in developer happiness entirely. And into safer code organization. This in turn will make feature additions possible which may increase the revenue.

So should you do it, too? I think you should either care for your code properly (and not allow it to mess up like I did, although unknowingly), or be honest with yourself and ditch the project when you wouldn't ever work on it anyway. I like to keep my projects healthy, so I have to [eat my own dogfood][explore].

[ttk]: https://github.com/DivineDominion/TestingToolkit
[explore]: /posts/2014/12/exploring-mac-app-development-strategies-released/
