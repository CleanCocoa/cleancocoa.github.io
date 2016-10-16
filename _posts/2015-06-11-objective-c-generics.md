---
title: Generics in Objective-C
created_at: 2015-06-11 16:50:10 +0200
kind: worklog
tags: [ objective-c, generics, safety ]
url: http://www.miqu.me/blog/2015/06/09/adopting-objectivec-generics/
comments: on
---

[Objective-C got generics!](http://www.miqu.me/blog/2015/06/09/adopting-objectivec-generics/) No more collection classes just for the sake of guarding against invalid elements!

I used custom collection classes a lot for the [Word Counter](http://wordcounterapp.com/) to let the compiler help me prevent mistakes. Swift generics were a godsend for that kind of safety. Now, Objective-C can achieve similar awesomeness.

It can look like this:

    @interface Wheel : NSObject
    @property (nonatomic, strong) UIColor *color;
    @end

    @interface Vehicle : NSObject
    @property (nonatomic, copy) NSArray<Wheel *> *wheels;
    @end

Now the compiler will warn (!) you if you misuse it:

    Vehicle *car = [[Vehicle alloc] init];
    [car.wheels addObject:@"NSString is of the wrong type"];

Great news, since if you didn't spend a lot of time creating custom collection types yourself, now at last can you ensure array element type consistency. Makes your code cleaner and more reliable.
