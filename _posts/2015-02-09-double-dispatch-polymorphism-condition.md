---
title: Use Double-Dispatch and Polymorphism to Get Rid of Conditionals
created_at: 2015-02-09 17:11:38 +0100
kind: worklog
tags: [ polymorphism, domain-model ]
vgwort: http://vg08.met.vgwort.de/na/aee9596ba7f24948ade9dacccd1f08c7
comments: on
code: objc
---

I want to point out a particular refactoring Jimmy Bogard shows in his video I mentioned [last time](/posts/2015/02/crafting-wicked-domain-models/).

Jimmy transforms a `switch` statement into a **double-dispatch** method call, making use of [good old **polymorphism**][polymorphism] to deal with variety of options. We'll see what these terms mean when applied.

Since your typical iOS application doesn't have a very intricate class hierarchy with lots of subclasses except those deriving from the frameworks's view controllers, thinking about the uses of polymorphism turns out to be a good exercise.

[Watch his presentation on Vimeo][pres] if you haven't already, because it's that cool.

Jimmy demonstrates how assigning offers to members of a coupon business became a one-liner in the actual service object. He got rid of all business logic in the service object. Instead, he pushed business logic into the Domain Model. This way, the Domain Model deserves to be called thus: the difference between a Domain Model and data containers which merely wrap the persistence layer is the _behavior_ of the Domain Model.

Let's have a look at an everyday `switch` conditional, then see what polymorphism can do to find an object-oriented alternative, and finally use double dispatch to put information in one place and behavior in another.


## Tell, Don't Ask: Push Business Logic Into Collaborators

Translated to Objective-C, Jimmy's example might have looked like this at some point:

    #!objectivec
    @implementation Member
    // ...
    
    - (void)assignOfferWithOfferType:(OfferType *)offerType 
                     offerCalculator:(id<CalculatesOffer>)offerCalculator {
        
        // How much is the discount?
        NSInteger value = [offerCalculator calculateValueFor:self 
                                                   offerType:offerType];
        
        // When does it expire?
        NSDate *expirationDate;
        NSTimeInterval daysValid = offerType.daysValid * 24 * 60 * 60;
        
        switch (offerType.expirationType)
        {
            case ExpirationTypeFixed:
                expirationDate = [NSDate dateWithTimeIntervalSinceNow:daysValid];
                break;

            case ExpirationTypeAssignment:
                expirationDate = [offerType.beginDate dateByAddingTimeInterval:daysValid];
                break;

            default:
                NSAssert(false, @"invalid expiration type");
                break;
        }
        
        Offer *offer = [[Offer alloc] initWithMember:self 
                                           offerType:offerType 
                                      expirationDate:expirationDate 
                                               value:value];
        
        [self.assignedOffers addObject:offer];
    }
    @end

Where `expirationType` property is an [enum][]:

    #!objectivec
    typedef NS_ENUM(NSInteger, ExpirationType) {
        ExpirationTypeFixed,
        ExpirationTypeAssignment
    };

This isn't too atypical. And it doesn't even look to bad or too involved.

Viewed from the outside, it's nice that `Member` takes care of creating its own `Offer` according to an `OfferType` and some calculation mechanism. When `Member` takes care of this, the client objects don't have to know how an `Offer` is created. Also, since `Offer` takes the member as an argument itself to form a bi-directional relationship, this eliminates mismatches altogether.

To make the method more readable and focused, you can get rid of the `switch` statement and write the whole method like this:

    #!objectivec
    - (void)assignOfferWithOfferType:(OfferType *)offerType 
                     offerCalculator:(id<CalculatesOffer>)offerCalculator {
        
        // How much is the discount?
        NSInteger value = [offerCalculator calculateValueFor:self 
                                                   offerType:offerType];
        
        // When does it expire?
        NSDate *expirationDate = [offerType calculateExpirationDate];
        
        Offer *offer = [[Offer alloc] initWithMember:self 
                                           offerType:offerType 
                                      expirationDate:expirationDate 
                                               value:value];
        
        [self.assignedOffers addObject:offer];
    }

Okay, the `switch` statement is now hidden inside `OfferType`'s `-calculateExpirationDate`. That shortens this method significantly and makes it easier to understand what's going on. Nothing special yet, though. 

What will `-calculateExpirationDate` look like? 

    #!objectivec
    - (NSDate *)calculateExpirationDate {
        
        return [self.expirationType calculateExpirationDateForOfferType:self];
    }

This isn't possible as long as `expirationType` is an `NSInteger`-based enum, though. It has to be a real object instead.

**First takeaway:** working with primitives is quick, but it also limits the amount of encapsulation when it comes to actual behavior.

## Polymorphism: Making `ExpirationType` a Set of Classes

To make `ExpirationType` a class but retain its enum-ness, we are going to define constants to make equality comparison possible:

    #!objectivec
    ExpirationType * const kExpirationTypeFixed = [ExpirationType expirationTypeFixed];
    ExpirationType * const kExpirationTypeAssignment = [ExpirationType expirationTypeAssignment];

    // Rename the enum to free the old name
    typedef NS_ENUM(NSInteger, ExpirationRule) {
        ExpirationRuleFixed,
        ExpirationRuleAssignment
    };

When we move the `switch` statement into  `-calculateExpirationDateForOfferType:offerType`, the naive first take will look like this:

    #!objectivec
    // ExpirationType.h
    @interface ExpirationType
    @property (assign) ExpirationRule expirationRule;
    + (instancetype)expirationTypeFixed;
    + (instancetype)expirationTypeAssignment;
    - (instancetype)initWithExpirationRule:(ExpirationRule)expirationRule;
    - (NSDate *)calculateExpirationDateForOfferType:(OfferType *)offerType;
    @end

    // ExpirationType.m
    @implementation ExpirationType
    + (instancetype)expirationTypeFixed {
        return [[self alloc] initWithExpirationRule:ExpirationRuleFixed];
    }
    
    + (instancetype)expirationTypeAssignment {
        return [[self alloc] initWithExpirationRule:ExpirationRuleAssignment];
    }
    
    // -initWithExpirationRule: is just what you'd expect: it sets the property
    
    - (NSDate *)calculateExpirationDateForOfferType:(OfferType *)offerType {
        
        NSTimeInterval daysValid = offerType.daysValid * 24 * 60 * 60;
        
        if (self.expirationRule == ExpirationRuleFixed)
            return [NSDate dateWithTimeIntervalSinceNow:daysValid];
        else if (self.expirationRule == ExpirationRuleAssignment)
            return [offerType.beginDate dateByAddingTimeInterval:daysValid];
        
        NSAssert(false, @"invalid expiration type");
        return nil;
    }
    @end

We've pushed logic down even another level. That's no gain in itself. It's just good to know that `ExpirationType` can determine its expiration date because that's a more natural place to look for the information.

The **double-dispatch** in here is in the flow of information there and back again: `OfferType` delegates calculation to its `ExpirationType` property (first "dispatch") and provides itself for further information. `ExpirationType` polls the `OfferType` to determine the time interval (second "dispatch", back to the originator).

Now we get to the fun part: using polymorphism to get rid of the `if` statement, too. Instead of `ExpirationType`, we will use concrete subclasses `FixedExpirationType` and `AssignmentExpirationType`. Objective-C doesn't know a thing about abstract classes or abstract methods. That's a good reason to not think in terms of abstract classes at all. But we will. And we'll hide the concrete subclasses from the outside world.

`ExpirationType` can get rid of a lot of functionality. It's just a better protocol. In fact, if it wasn't for the convenience method `-daysValidTimeInterval:` and the factory methods, you can perfectly model this as a protocol:

    #!objectivec
    // ExpirationType.h
    @interface ExpirationType
    - (NSDate *)calculateExpirationDateForOfferType:(OfferType *)offerType;
    @end

    // ExpirationType.m    
    @interface FixedExpirationType : ExpirationType
    @end
    
    @interface AssignmentExpirationType : ExpirationType
    @end
    
    @implementation ExpirationType
    + (instancetype)expirationTypeFixed {
        return [[FixedExpirationType alloc] init];
    }
    
    + (instancetype)expirationTypeAssignment {
        return [[AssignmentExpirationType alloc] init];
    }
    
    - (NSDate *)calculateExpirationDateForOfferType:(OfferType *)offerType {
        
        NSAssert(false, @"override this method in concrete subclasses");
        return nil;
    }
    
    - (NSTimeInterval)daysValidTimeInterval:(OfferType *)offerType {
        
        return offerType.daysValid * 24 * 60 * 60;
    }
    @end

Place the concrete subclasses in the `.m` file, too, so no-one else knows about them:
    
    #!objectivec
    @implementation FixedExpirationType
    - (NSDate *)calculateExpirationDateForOfferType:(OfferType *)offerType {
        
        return [NSDate dateWithTimeIntervalSinceNow:daysValid];
    }
    @end
    
    @implementation AssignmentExpirationType
    - (NSDate *)calculateExpirationDateForOfferType:(OfferType *)offerType {
        
        return [offerType.beginDate dateByAddingTimeInterval:daysValid];
    }
    @end

Suddenly, there's no else-clause anymore, which means one whole reason less for the app to fail. There can only be two kinds of `ExpirationType`. You don't even need the enum right now. Even the constants are optional: it's okay to deal with more than one instance of `FixedExpirationType`, for example, because it only performs a simple method anyway.

You will need some kind of `ExpirationRule` equivalent to persist the type of expiration. Maybe add another "abstract" method `-toInteger` which returns 0 for `FixedExpirationType` and 1 for `AssignmentExpirationType`.

Next, I'd move what `-daysValidTimeInterval` does into a `Days` Value Type to put the computation where it belongs: certainly not in the client of `offerType.daysValid`.

I think this is a great reminder why polymorphism is useful, and how some things can't be solved elegantly by delegation alone.

**Second takeaway:** You can replace `switch` and `if` statements with polymorphism.

[Watch the video][pres] or read a [summary][] to see the rest of Jimmy's insights.


[enum]: http://nshipster.com/ns_enum-ns_options/
[pres]: http://vimeo.com/43598193
[polymorphism]: http://en.wikipedia.org/wiki/Polymorphism_%28computer_science%29
[summary]: http://programmers.stackexchange.com/a/221238/89665
