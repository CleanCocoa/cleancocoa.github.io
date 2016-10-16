---
title: Small Helper Objects Take You Far
created_at: 2015-10-24 11:14:45 +0200
kind: worklog
tags: [ clean, objective-c, refactoring ]
shared_image: "blog/201509021531_clean-up-t.jpg"
comments: on
---

Integrating new functionality is fun. But revisiting 18-months old code isn't. Back then I created a protocol `InvokesWindows` to define methods like `-showPreferencesWindow` which I imported in the menu bar controller to show the preferences when the user selects a pop-up menu item. _But_ I didn't actually delegate to any instance of `InvokesWindows`. I used `NSApp`. (Insert facepalm here.)

Which version do you like better?

    #!objc
    - (void)preferencesMenuItemSelected:(id)sender
    {   
        [NSApp sendAction:@selector(showPreferencesWindow) to:nil from:self];
    }

    - (void)preferencesMenuItemSelected:(id)sender
    {
        [self.router showPreferences];
    }

Well, the `NSApp`-thing helped me move quickly without creating another two files for an actual "window invoker" in Objective-C. Coming back to this code base from my Swift projects of the past months, I feel super-slow. Writing Objective-C feels weird again.

As [Matt Kremer](https://mattkremer.com/embrace-your-developer-personality/) recently blogged, there's two phases of a project, and two developer personalities which fit each phase best: 

1. there's a growing phase of the project where all functionality is initially developed; it's great to make progress fast and ship early
2. there's a maintenance phase where you have problems understanding the shortcuts you or a team mate took; this is where lots of people begin to clean up

The new `router` property is the result of me maintaining the old code base. Back then, I was worried about other things and didn't put "refactor NSApp calls away" onto my running to-do list, it seems.

`InvokesWindows` is now gone in favor of `RoutesModules`, a name which captures the intent much better, and which is actually used:

    #!objc
    @protocol CTWRoutesModules <NSObject>
    - (void)showHistory;
    - (void)showFileMonitoring;
    - (void)showPreferences;
    - (void)showLicense;
    @end

The actual implementation is very, very simple. And it's written in Swift.

    #!swift
    import Foundation
    import FileMonitoringModule

    @objc(AppModuleRouter)
    class AppModuleRouter: NSObject, CTWRoutesModules {
    
        private var history: CTWShowHistory?
    
        func showHistory() {
        
            history = CTWShowHistory()
            history?.showHistoryWindow()
        }
    
        func showFileMonitoring() {
        
            FileMonitoringModule.showFileMonitoringWindow()
        }
    }

Back then I created a "Show History" use case object which I still stick to. But the new file monitoring module is an actual module, a Swift Framework. And for it I created another framework which bootstraps all components with default values for the actual application -- this takes care of itself and offers a single facade to all underlying functionality: 

    #!swift
    private let bootstrapping = BootstrapFileMonitoring()
    private var bootstrapping_token: dispatch_once_t = 0

    @objc(FileMonitoringModule)
    public class FileMonitoringModule: NSObject {
        
        public static func bootstrapModule() {

            dispatch_once(&bootstrapping_token) {
                bootstrapping.bootstrapModule()
            }
        }
    
        public static func showFileMonitoringWindow() {
        
            bootstrapping.showFileMonitoringWindow()
        }
    }

I write simpler code nowadays. I think Swift taught me some tricks I haven't come up with in Objective-C. Also I learned a lot more about software architecture in the past year.

I am not getting rid of most calls to `NSApp`. Not because it didn't work or because there are too many of them -- it's because the result is easier to read and understand. And I can finally unit test the actual routing by injecting a test double.

---

Photo Credit: [Please Clean Up Your Mess](https://www.flickr.com/photos/allen_goldblatt/194069411) by [Allen Goldblatt](https://www.flickr.com/photos/allen_goldblatt/). License: [CC-BY-2.0](https://creativecommons.org/licenses/by/2.0/).
