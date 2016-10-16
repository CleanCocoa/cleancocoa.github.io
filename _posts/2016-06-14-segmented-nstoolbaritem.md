---
title: How to Create a Segmented NSToolbarItem like Apple Mail.app
created_at: 2016-06-14 15:34:06 +0200
kind: worklog
tags: [ toolbar, appkit ]
image: 201606141533_window.png
vgwort: http://vg01.met.vgwort.de/na/81ea1899bca84722bf9e23a371781d45
comments: on
---

Ever wondered how Apple Mail's `NSToolbar` achieves the effect of individually labeled buttons shown in groups? 

{% include figure.md src="/assets/blog/2016/201606141536_mail.png" alt="mail toolbar screenshot" caption="Screenshot of Mail on OS X 10.11" %}

If you know your AppKit, you'll try to drag a `NSSegmentedControl` into the toolbar and fiddle around with it. But there'll only be _one_ label for the entire `NSSegmentedControl`.

The solution is simple but weird: there's a `NSToolbarItem` subclass called `NSToolbarItemGroup` which can group a collection of `NSToolbarItem`s so they can be moved around together only. In combination with a `NSSegmentedControl` you can label each segment individually.

You have to get out of Interface Builder for this and customize your `NSToolbarDelegate` to return a `NSToolbarItemGroup` instance:

    #!swift
    let group = NSToolbarItemGroup(itemIdentifier: "NavigationToolbarItems")

    // First add subitems
    let itemA = NSToolbarItem(itemIdentifier: "PrevToolbarItem")
    itemA.label = "Prev"
    
    let itemB = NSToolbarItem(itemIdentifier: "NextToolbarItem")
    itemB.label = "Next"

    group.subitems = [itemA, itemB]
    
    // ... then add a custom `view`
    let segmented = NSSegmentedControl()
    group.view = segmented


The weird thing about this is that you add _both_ one subitem for each segment _and_ the `NSSegmentedControl` as the group's own view.

The segment labels will be obtained from the `NSToolbarItem`s. Don't set a label for the segments themselves (via `NSSegmentedControl.setLabel(_:, forSegment:)`) -- these will be drawn inside the controls.

## Fully Working Sample App

{% include figure.md src="/assets/blog/2016/201606141533_window.png" alt="sample app screenshot" caption="The resulting sample toolbar" %}

1. Create a new OS X/Cocoa project
2. Paste the following code into `AppDelegate.swift`
3. There is no step 3

<!-- separator -->

    #!swift
    import Cocoa

    @NSApplicationMain
    class AppDelegate: NSObject, NSApplicationDelegate, NSToolbarDelegate {

        @IBOutlet weak var window: NSWindow!

        var toolbar: NSToolbar!

        let toolbarItems: [[String: String]] = [
            ["title" : "irrelevant :)", "icon": "NSPreferencesGeneral", "identifier": "NavigationGroupToolbarItem"],
            ["title" : "Share", "icon": NSImageNameShareTemplate, "identifier": "ShareToolbarItem"],
            ["title" : "Add", "icon": NSImageNameAddTemplate, "identifier": "AddToolbarItem"]
        ]

        var toolbarTabsIdentifiers: [String] {

            return toolbarItems.flatMap { $0["identifier"] }
        }

        func applicationDidFinishLaunching(aNotification: NSNotification) {

            toolbar = NSToolbar(identifier: "TheToolbarIdentifier")
            toolbar.allowsUserCustomization = true
            toolbar.delegate = self
            self.window?.toolbar = toolbar
        }

        func toolbar(toolbar: NSToolbar, itemForItemIdentifier itemIdentifier: String, willBeInsertedIntoToolbar flag: Bool) -> NSToolbarItem? {

            guard let infoDictionary: [String : String] = toolbarItems.filter({ $0["identifier"] == itemIdentifier }).first
                else { return nil }

            let toolbarItem: NSToolbarItem

            if itemIdentifier == "NavigationGroupToolbarItem" {

                let group = NSToolbarItemGroup(itemIdentifier: itemIdentifier)

                let itemA = NSToolbarItem(itemIdentifier: "PrevToolbarItem")
                itemA.label = "Prev"
                let itemB = NSToolbarItem(itemIdentifier: "NextToolbarItem")
                itemB.label = "Next"

                let segmented = NSSegmentedControl(frame: NSRect(x: 0, y: 0, width: 85, height: 40))
                segmented.segmentStyle = .TexturedRounded
                segmented.trackingMode = .Momentary
                segmented.segmentCount = 2
                // Don't set a label: these would appear inside the button
                segmented.setImage(NSImage(named: NSImageNameGoLeftTemplate)!, forSegment: 0)
                segmented.setWidth(40, forSegment: 0)
                segmented.setImage(NSImage(named: NSImageNameGoRightTemplate)!, forSegment: 1)
                segmented.setWidth(40, forSegment: 1)

                // `group.label` would overwrite segment labels
                group.paletteLabel = "Navigation"
                group.subitems = [itemA, itemB]
                group.view = segmented

                toolbarItem = group
            } else {
                toolbarItem = NSToolbarItem(itemIdentifier: itemIdentifier)
                toolbarItem.label = infoDictionary["title"]!

                let iconImage = NSImage(named: infoDictionary["icon"]!)
                let button = NSButton(frame: NSRect(x: 0, y: 0, width: 40, height: 40))
                button.title = ""
                button.image = iconImage
                button.bezelStyle = .TexturedRoundedBezelStyle
                toolbarItem.view = button
            }

            return toolbarItem
        }

        func toolbarDefaultItemIdentifiers(toolbar: NSToolbar) -> [String] {

            return self.toolbarTabsIdentifiers;
        }

        func toolbarAllowedItemIdentifiers(toolbar: NSToolbar) -> [String] {

            return self.toolbarDefaultItemIdentifiers(toolbar)
        }

        func toolbarSelectableItemIdentifiers(toolbar: NSToolbar) -> [String] {

            return self.toolbarDefaultItemIdentifiers(toolbar)
        }

        func toolbarWillAddItem(notification: NSNotification) {

            print("toolbarWillAddItem", (notification.userInfo?["item"] as? NSToolbarItem)?.itemIdentifier ?? "")
        }

        func toolbarDidRemoveItem(notification: NSNotification) {

            print("toolbarDidRemoveItem", (notification.userInfo?["item"] as? NSToolbarItem)?.itemIdentifier ?? "")

        }

        func applicationWillTerminate(aNotification: NSNotification) {
            // Insert code here to tear down your application
        }
    
    }

**Update 2016-06-16:** Enabling/disabling segments in the toolbar takes a bit of extra work. In short, you have to both disable the segment through the `NSSegmentedControl` and the corresponding `NSToolbarItem`. I wrote [a follow-up post about disabling segments](content/posts/2016/06/disabling-toolbar-item-segments.txt).
