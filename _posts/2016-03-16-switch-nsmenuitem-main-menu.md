---
title: How I Toggle or Switch 2 NSMenuItems from the Main Menu
created_at: 2016-03-16 12:55:21 +0100
kind: worklog
tags: [ menu, 1df, nsdocument ]
comments: on
---

I thought it'd be straigthforward and simple to make a `NSMenuItem` from the main menu implement a toggle -- be it a checkmark or switching "Show X" with "Hide X" conditionally.

Turns out that's not quite as simple as I had hoped. Cocoa bindings would work but [make things complicated](http://www.cocoabuilder.com/archive/cocoa/209750-binding-nsmenuitem-state-in-document-based-app.html#209771). Most stuff on the web uses view tags to find items in menus. That's not my favorite solution for anything. `menuNeedsUpdate` wasn't called when I had hoped it would, either. So I tried a few different setups and settled with a boring and verbose way to switch "Show X" and "Hide X" depending on a boolean flag the current `NSDocument` window exposes.

## Setup the main menu items

The [human interface guidelines](https://developer.apple.com/library/mac/documentation/UserExperience/Conceptual/OSXHIGuidelines/MenuChanging.html#//apple_ref/doc/uid/20000957-CH26-SW1) favor showing both "Show" and "Hide" but disabling one of them over checkmarks because checkmarks can be confusing for some users. The next best solution to save menu item space -- and one we're accustomed to -- is to replace the "Show" menu item with the "Hide" menu item and vice versa.

Replacing can mean two things: actually inserting and removing items, or hiding and showing them conditionally. I chose setting the `hidden` flag.

{% include figure.md src="/assets/blog/2016/1-hidden-item.png" alt="screenshot of hidden item" caption="Hide the alternate item by default" %}

## Create a controller and its outlets

A very old-fashioned way without any bells and whistles is to set both menu items as `IBOutlet`s of some controller. The window controller isn't a good place because there are potentially many at the same time in `NSDocument`-based apps. The `AppDelegate` comes to mind, but better not clutter it with details.

So I create a `RowNumbersMenuToggle` type and connect the menu items to its outlets. At first, the class looks very simple:

    #!swift
    class RowNumbersMenuToggle: NSObject {

        @IBOutlet var showRowNumbersMenuItem: NSMenuItem!
        @IBOutlet var hideRowNumbersMenuItem: NSMenuItem!

        func toggleShowsRowNumbers(showingRowNumbers: Bool) {

            if showingRowNumbers {
                enableHideRowNumbers()
            } else {
                enableShowRowNumbers()
            }
        }

        func enableHideRowNumbers() {

            showRowNumbersMenuItem.hidden = true
            hideRowNumbersMenuItem.hidden = false
        }

        func enableShowRowNumbers() {

            showRowNumbersMenuItem.hidden = false
            hideRowNumbersMenuItem.hidden = true
        }
    }

In the screenshot from above, you may have noticed I added this controller as an object to the `MainMenu.nib` already. It's created alongside the main menu upon launch. I use the `AppDelegate` as the owner for now to retain the object.

    #!swift
    @NSApplicationMain
    class AppDelegate: NSObject, NSApplicationDelegate {

        @IBOutlet var rowNumbersMenuToggle: RowNumbersMenuToggle!
        
        // ...
    }


## Listen to window key events

In order to conditionally display the appropriate menu items, the frontmost `NSDocument`'s window controller will be queried for its `showsRowNumbers` state. 

Since `NSWindowController`s oddly don't have a `windowWillClose` callback or similar display state callbacks we know from `UIViewController`, we have to rely on notifications. There are two useful notifications for this:

* `NSWindowDidBecomeKeyNotification` to find the newly activated window, and
* `NSWindowDidResignKeyNotification` to deactivate the current window.

The window controllers I use are of type `TableWindowController`. If the key window is not one of that kind, reset the `RowNumbersMenuToggle` state.

    #!swift
    class RowNumbersMenuToggle: NSObject {

        lazy var notificationCenter: NSNotificationCenter = NSNotificationCenter.defaultCenter()

        var didBecomeKeyObserver: NSObjectProtocol?
        var didResignKeyObserver: NSObjectProtocol?

        override func awakeFromNib() {

            super.awakeFromNib()

            didBecomeKeyObserver = notificationCenter.addObserverForName(
                NSWindowDidBecomeKeyNotification, object: nil, queue: nil) { [weak self] notification in

                guard let window = notification.object as? NSWindow, 
                    controller = window.delegate as? TableWindowController else {

                    self?.keyWindowController = nil
                    return
                }

                self?.keyWindowController = controller
            }

            didResignKeyObserver = notificationCenter.addObserverForName(
                NSWindowDidResignKeyNotification, object: nil, queue: nil) { [weak self] notification in

                guard let window = notification.object as? NSWindow, 
                    controller = window.delegate as? TableWindowController else {

                    return
                }

                self?.keyWindowController = nil
            }
        }
        
        weak var keyWindowController: TableWindowController?
    }

All this boils down to setting the `keyWindowController` property when the key window changes.

I don't like that I have hard-wired knowledge about the key window's delegate being `TableWindowController` into this class. But for now that's bearable. It's a pretty simple setup and there won't be much (if any) deviation from this pattern. It's introducing implicit tight coupling here, but at a very reasonable cost.

## Connect the key window state to the menu items

Things get interesting with property observers:

    #!swift
    weak var keyWindowController: TableWindowController? {

        willSet {

            keyWindowController?.showsRowNumbersHandler = nil
        }

        didSet {

            guard let controller = keyWindowController else {
                return
            }

            keyWindowController?.showsRowNumbersHandler = { [weak self] showsRowNumbers in
                self?.toggleShowsRowNumbers(showsRowNumbers)
            }

            toggleShowsRowNumbers(controller.showsRowNumbers)
        }
    }

The window controller exposes a `showsRowNumbersHandler` callback of type `(Bool) -> Void` this object hooks in to.

So all in all four things are involved:

1. A custom type with outlets for the menu items you want to switch (here: `RowNumbersMenuToggle`),
2. an objects that retains this controller (here: `AppDelegate`),
3. the MainMenu nib where the connections are created,
4. the `NSDocument`'s window controller to report the interesting view state.

It's not the sexiest of all approaches but it's pretty solid as is.

## Improving this clumsy approach

I see two ways to improve this:

1. Emit events instead. In the spirit of `#1DF`/unidirectional flow or ReSwift, changing the key window emits a global event (which a menu controller subscribes to) and using the menu item emits an event to the key document's store to toggle the view state. No back-and-forth subscribing.
2. Create an abstraction of this. In the spirit of [Behaviors](/posts/2016/03/controller-behavior-extension/), create a general `MenuItemsToggle` for each pair of items. Connect to it from an object that invokes the toggle method or bind it to a boolean to let it change on its own.

The problem with unidirectional flow is that the main menu in general is both independent from _and_ depending on the frontmost `NSDocument`. That means the menu items are toggled depending on the document's state store. But only _some_. And the notion of which document is key is not necessarily part of the document's state but rather of the app itself. I bet I'd have to model this at least twice to get it working in a decent manner.

Overall, this is so involved for such a simple task that I wonder if "put stuff into AppDelegate" couldn't be a design pattern after all ... Any other suggestions?
