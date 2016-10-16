---
title: Disabling Segments in a NSSegmentedControl in a Toolbar
created_at: 2016-06-16 19:01:19 +0200
kind: worklog
tags: [ toolbar, appkit ]
vgwort: http://vg01.met.vgwort.de/na/a47ead4ad7cb4c9caae129df20c0990d
comments: on
---

Earlier this week I posted [how to create a segmented toolbar item with 1 label for each segment][seg]. Now some options in TableFlip depend on context: without a selected cell, you cannot remove the selection, for example. So I disabled some segments and it looked alright:

{% include figure.md src="/assets/blog/2016/201606161900_disabled.png" alt="disabled segments screenshot" caption="How the segments should look with 2/3 disabled" %}

Easy enough: 

    #!swift
    let enabled = hasSelection()

    ThisParticularSegmentedControl.contextualSegments // = (0...1)
        .forEach { (segment: Int) in
            self.setEnabled(enabled, forSegment: segment)
    }

But even though it's disabled, and even though the `NSSegmentedControl`'s `trackingMode` is set to `NSSegmentSwitchTracking.Momentary`, clicking a disabled segment in the toolbar results in a permanent disabled-looking selection:

{% include figure.md src="/assets/blog/2016/201606161901_selected.png" alt="selected disabled segments screenshot" caption="Disabled but toggled segment" %}

I tried a _ton_ of different things, including, but not limited to:

* when the selection changes, deselect the segment again
* when the toolbar validates, set `selectedSegment = -1` ("no selection")

The problem isn't with the `NSSegmentedControl`, though. It's with the `NSToolbarItem` that corresponds to the disabled segments!

So during `NSToolbarItemGroup.validate()` in my custom subclass I disable the corresponding subitems, too.

## Some Example Code

This is the group:

    #!swift
    class ToolbarItemGroup: NSToolbarItemGroup {

        convenience init(itemIdentifier: String, validator: ActionItemValidation) {

            self.init(itemIdentifier: itemIdentifier)

            self.validator = validator
        }

        var validator: ActionItemValidation!

        override func validate() {

            validateSubitems()
            validateSegmentedControl()
        }

        private func validateSubitems() {

            self.subitems.forEach(validator.validate)
        }

        private func validateSegmentedControl() {

            guard let segmentedControl = self.view as? AddDimensionSegmentedControl 
                else { return }

            validator.validate(segmentedControl: segmentedControl)
        }
    }

The `ActionItemValidation` type exposes two relevant methods:

1. `validate(segmentedControl:)`
2. `validate(toolbarItem:)`

The `validate(toolbarItem:)` variant existed already to enable/disable buttons in the toolbar. I achieve this by tagging the items, for example: 

* `0` equals "always on", 
* `1` equals "enable only for selections".

Re-using the method here was easy. I only had to set the tag for the group's affected subitems.

[seg]: /posts/2016/06/segmented-nstoolbaritem/
