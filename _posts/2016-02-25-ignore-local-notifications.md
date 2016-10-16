---
title: Ignoring UILocalNotifications When the App is Running
created_at: 2016-02-25 18:44:07 +0100
kind: worklog
tags: [ notification ]
comments: on
---

Maybe my Google-fu is lacking or there's not much about this on the web: how do you separate handling local notifications when the app is in foreground from when it is in background? After all you'll probably want to show a specific view when the user taps a notification badge. Here's one way to do it.

When you schedule `UILocalNotification`s for a future time and when the user is using the app at that moment, `application(_, didReceiveLocalNotification:)` gets called -- the same `AppDelegate` callback that'll be invoked when the user taps on a notification badge while your app is in background mode. I found that surprisingly inconvenient.

The only notification the app I'm working on receives is a reminder. When the user taps the badge, the app activates and the event details should appear. When the app is in foreground already, nothing should happen. The user should not be magically taken to the event detaila view while she's doing something elss. How weird would that be?

So we need a toggle to ignore the notification when the app is open already.

    #!swift
    func application(application: UIApplication, didReceiveLocalNotification notification: UILocalNotification) {
        guard !isActive else { return }
    
        handleLocalNotification(localNotification)
    }

Where does the `isActive` toggle gets set, though?

A naive approach is to set it to `false` when the app goes into background mode. Only the app comes into foreground _and then_ handles the notification when you interact with the badge, so that's not a good fit. Active/inactive callbacks work better since `applicationDidBecomeActive(_)` is called last, for whatever reason.

    #!swift
    var isActive = true
    
    func applicationDidBecomeActive(application: UIApplication) {
        isActive = true
    }
    
    func applicationWillResignActive(application: UIApplication) {
        isActive = false
    }

This works as expected and we're set.

Only then did I discover `UIApplicationState`. We can get rid of all the toggle-related code and write this instead:

    #!swift
    func application(application: UIApplication, didReceiveLocalNotification notification: UILocalNotification) {
        guard application.applicationState == .Inactive else { return }
    
        handleLocalNotification(localNotification)
    }

Sheesh.
