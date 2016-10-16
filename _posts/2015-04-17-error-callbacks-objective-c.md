---
title: "Success and Error Callbacks in Objective-C: a Refactoring"
created_at: 2015-04-17 11:54:50 +0200
kind: worklog
tags: [ objective-c, error-handling, refactoring ]
comments: on
---

I'm refactoring code of [Calendar Paste 2](http://calendarpasteapp.com) some more. Since I like what I learned using Swift so much, one of today's changes was about making a view controller method do _less_ by factoring object creation and error handling into its collaborators.

The resulting code handles far less of the action involved. Instead, it delegates the action to a new [command-like object](http://en.wikipedia.org/wiki/Command_pattern). And instead of querying an event template for data and assembling an actual event with start and end dates, it delegates event creation to the template itself. _Tell, Don't Ask_ saved the day.

Here's the old code for using an event template and paste it into a calendar:


    #!obj-c
    - (void)paste:(id)sender
    {
        EKEventStore *eventStore = self.shift.eventStore;
        EKEvent *event           = [self.shift event];
    
        event.allDay = [self.shift isAllDay];
    
        event.startDate = self.startDate;
        event.endDate   = [self endDate];
    
        NSError *error = nil;
    
        BOOL success = [eventStore saveEvent:event span:EKSpanThisEvent commit:YES error:&error];
        NSAssert(success, @"saving the event failed: %@", error);
    
        if (success)
        {
            if ([self.shift isAllDay] == NO)
            {
                [self.shift setLastPaste:self.startDate];
                [self.shiftTemplateController saveManagedObjectContext];
            }
        
            [self performSegueWithIdentifier:kSegueUnwindPasted sender:sender];
        }
        else
        {
            [self performSegueWithIdentifier:kSegueUnwindCancelPasting sender:sender];
        }
    }

It already uses segues, but it's still reaching deep into `shift` to access the `eventStore` and save the actual event.

The shift itself isn't too well-factored at the moment. I don't want to put saving actual events into event stores in there as well. But the shift has knowledge about all necessary parts, so it's only natural to create a factory on it:

    #!obj-c
    - (PreparedEvent *)preparedEventWithStartDate:(NSDate *)startDate 
                                          endDate:(NSDate *)endDate;

I create `PreparedEvent` to hold on to the `event` and `eventStore`, both of which are known to a `shift`. Then it wraps the saving command with callbacks:

    #!obj-c
    - (void)saveCompletionHandler:(void (^)())successCallback onError:(void (^)(NSError *))errorCallback
    {
        EKEventStore *eventStore = self.eventStore;
        EKEvent *event = self.event;
    
        NSError *error = nil;
        BOOL success = [eventStore saveEvent:event span:EKSpanThisEvent commit:YES error:&error];
        NSAssert(success, @"saving the event failed: %@", error);
    
        if (success) {
            successCallback();
        } else {
            errorCallback(error);
        }
    }

That's making `ShiftAssignmentViewController -paste:` way leaner:
    
    #!obj-c
    - (void)paste:(id)sender
    {
        ShiftTemplate *shift = self.shift;
        NSDate *startDate = self.startDate;
        NSDate *endDate = self.endDate;
    
        PreparedEvent *event = [shift preparedEventWithStartDate:startDate endDate:endDate];
    
        [event saveCompletionHandler:^{
            if ([shift isAllDay] == NO)
            {
                [shift setLastPaste:self.startDate];
                [self.shiftTemplateController saveManagedObjectContext];
            }
        
            [self performSegueWithIdentifier:kSegueUnwindPasted sender:sender];
        } onError: ^(NSError *error){
            [self performSegueWithIdentifier:kSegueUnwindCancelPasting sender:sender];
        }];
    }

Both a success and error callback in a single method call don't read too well in Objective-C, but I prefer this to the back-and-forth I had before.

**Have you used a similar pattern in your code? Or do you stick to inout `NSError` parameters?**
