---
title: Dismissing a Modally Presented Scene on Both iPad and iPhones Using Unwind Segues
created_at: 2015-09-14 17:12:28 +0200
kind: worklog
tags: [ storyboard, segue, uiviewcontroller, objective-c, uikit ]
comments: on
---

`UISplitViewController` is the way to go when you want to make your iOS app universal without much hassle and can model the scenes in terms of master/detail.

While getting [Calendar Paste][cpapp] ready for the upcoming iOS 9 release, I discovered that using `UISplitViewController` across devices is one thing, while Storyboard segues are another.

Unwinding a modally presented scene on the regularly small iPhones (or "compact" size class phones) in a Storyboard involves dismissing the presented view controller. On the iPad, though, the `- (IBAction)unwindSegue:` callback will be used _while the presented view controller is not dismissed._

StackOverflow [knows about this issue](http://stackoverflow.com/questions/28247727/unwind-segue-doesnt-dismiss-adaptive-popover-presentation-when-not-modal), so it seems not to be only me.

To avoid unbalanced or self-interrupting transitions, I have made my main view controller state-ful. It has a `performAfterTransition` property. When there's a block attached to it, it will execute and reset itself in `viewDidAppear`: that's a hook for compact size iPhones. The underlying or presenting scene will "appear" after the modally presented scene is dismissed.

Since the iPad doesn't dismiss presented scenes automatically when unwinding to the view controller shown a layer beneath the modal scene, you have to call `dismissViewControllerAnimated:completion:` manually.

I encapsulated the checks in little helper methods.

    #!objc
    - (IBAction)unwindSegue:(UIStoryboardSegue *)segue
    {
        if ([self unwindsFromEmptyList:segue])
        {
            UIViewController *sourceVC = segue.sourceViewController;
            __weak id welf = self;
        
            [self executeClosure:^{
                [welf displayAddTemplateScene];
            } afterDismissing:sourceVC];
        }
        // ...
    }
    
    - (BOOL)unwindsFromEmptyList:(UIStoryboardSegue *)segue
    {
        return [segue.identifier isEqualToString:kSegueUnwindAddFirstTemplate];
    }

So here they are, hiding the iPad/iPhone distinction in their bellies:

    #!objc
    - (void)executeClosure:(void (^)(void))closure afterDismissing:(UIViewController *)viewController
    {
        // This check and the call to `dismissViewControllerAnimated:completion:`
        // was added thanks to iPad.
        if (!viewController.isBeingDismissed)
        {
            [viewController dismissViewControllerAnimated:YES completion:closure];
        } else {
        
            // This is how I have done it on the iPhone originally.
            [self executeClosureAfterUnwinding:closure];
        }
    }

    - (void)executeClosureAfterUnwinding:(void (^)(void))closure
    {
        self.performAfterTransition = closure;
    }
    
And this is how I execute the post-transition hooks. They are necessary to automatically cover the main list with full-screen notifications on first launch. You are discouraged to perform a transition while another one is still active, or so the logs indicate. `viewDidAppear:` sounded like a good hook to separate the transitions. And it works.

    #!objc
    - (void)viewDidAppear:(BOOL)animated
    {
        [super viewDidAppear:animated];

        [self performPostTransitionClosure];
    }

    - (void)performPostTransitionClosure
    {
        Closure closure = self.performAfterTransition;
    
        if (closure)
        {
            closure();
            self.performAfterTransition = nil;
        }
    }
    
[cpapp]: http://calendarpasteapp.com
