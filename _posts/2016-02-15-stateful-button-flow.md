---
title: Information Flow and Stateful Buttons
created_at: 2016-02-15 11:04:37 +0100
kind: worklog
tags: [ reswift, oop, 1df ]
comments: on
---

After reading [Sam's post](/posts/2016/02/realm-unidirectional-flow/), I discussed a code snippet with him. Shortsightedly, I told him I didn't like the following snippet because it controls `project` instead of doing the object its own job:

    #!swift
    @IBAction func activityButtonTapped() {
        guard let project = project else { return }
        if project.currentActivity == nil {
            store.startActivity(project, startDate: NSDate())
        } else {
            store.endActivity(project, endDate: NSDate())
        }
    }

It made me wonder if we can do better, though, with standard OOP practice. My proposal was a simple toggle method:

    #!swift
    @IBAction func activityButtonTapped() {
        guard let project = project else { return }
        project.toggleActivity(store: store, now: NSDate())
    }

Sure, this respects the objects boundaries and lets it control itself. I thought we were done.

Sam pointed out that we may introduce race conditions, though: if the user has 2 devices and taps the "start" button on both while they synced state, device 1 will toggle to start the activity tracker and device 2 will toggle to end it right away. The user's intention is to start it on both devices, so a toggle may be the wrong approach.

The problem with `activityButtonTapped()` is that it _is_ a toggle. To get rid of that and respect the project's boundaries, we would ideally introduce two action callbacks:

    #!swift
    @IBAction func startButtonTapped() {
        guard let project = project else { return }
        
        store.startActivity(project, startDate: NSDate())
    }
    
    @IBAction func stopButtonTapped() {
        guard let project = project else { return }
        
        store.endActivity(project, endDate: NSDate())
    }

Now the UI has to utilize them both -- either it has to display two buttons and (de)activate one of them conditionally, or it switches the actions on a single button. 

This works great with uni-directional flow of information. If the state is changed and the UI is updated with the new state, the correct button will be activated. Race conditions like those Sam pointed out to me will be gone. If the user taps "start" on two devices, the state of both will reflect a started activity.

[rs]: /posts/2016/01/reswift-level-indirection/
