---
title: "Fixing NSTableView Cell Backgrounds by Deactivating NSSplitViewItem's Visual Effects"
created_at: 2017-06-17 13:23:03 +0200
tags: [ nstableview, nssplitview ]
vgwort: http://vg01.met.vgwort.de/na/e73f21a7acb34205b0ac61afa3b46a01
comments: on
---

[I discovered my own idiocy](/posts/2017/06/18/nssplitviewitem-kinds/) and found that I overlooked which initializer of `NSSplitViewItem` was called. Apparently, the sidebar-related one adds vibrancy by default. Dealing with un-vibrancifying a table view might still be interesting, so I leave this up.

Don't jump to conclusions, folks, and don't copy faulty code over to test projects: if I had typed the `NSSplitViewItem`s setup code instead of pasting it in, I would've caught the initializer's parameter difference.

---------

My table view items looked odd for a while and I couldn't figure out why. Until I disabled "Reduce transparency" in my System Preferences's "Accessiblity" pane. 

{% include figure.md src="/assets/blog/2017/20170617130405_tableview-backgrounds.png" alt="screenshot of broken table views" caption="The various kinds of weird background artifacts that turned out to be behind-window blendings." %}

This was 0% my own genius and 100% thanks to a beta tester. (Thanks, Michel!) 

I couldn't reproduce this in a simple test app, no matter the settings of the views. Until I looked at the implementation of the split view controller. In the real app, I manage the split view with a `NSSplitViewController` subclass. Using this controller class, all of a sudden `NSSplitViewItem`s are being wrapped in `_NSSplitViewItemViewWrapper`, each containing a `NSVisualEffectsView`. The dividers become "vibrant", too.

`NSSplitViewController` only wraps the main pane in a visual effects view.**  If you have a single pane, that will be wrapped. If you have 2 or more, only  the pane at index #1 will be wrapped. Sidebars to the left and right are not affected, no matter how many you have. This assumed #1 is not a sidebar, of course.

Here's the list of subviews of the `NSSplitView` when managed by a `NSSplitViewController`:

```
▿ 7 elements
  - 0 : <_NSSplitViewItemViewWrapper: 0x6000001a46e0>
  - 1 : <_NSSplitViewItemViewWrapper: 0x6080001a32c0>
  - 2 : <_NSSplitViewItemViewWrapper: 0x6080001a3480>
  - 3 : <NSVibrantSplitDividerView: 0x6080001863f0>
  - 4 : <NSVibrantSplitDividerView: 0x6080001864c0>
  - 5 : <_NSSplitViewSpringLoadingView: 0x608000169540>
  - 6 : <_NSSplitViewSpringLoadingView: 0x608000169600>
```

Debugging the UI shows this in the view hierarchy breadcrumb list:

{% include figure.md src="/assets/blog/2017/20170617123914_nssplitview-hierarchy.png" alt="UI hierarchy screenshot" caption="Actual and expected view hierarchy" %}

When you don't use `NSSplitViewController` and its `insertSplitViewItem(_:at:)` or `addSplitViewItem(_:)`, the visual effect views won't be added. As expected. There's no documented opting-out of this, either.

> The first step in supporting vibrancy in a view is to contain the view in an instance of NSVisualEffectView. It is typically best to add vibrancy only to leaf views, not container views, because it is difficult to turn vibrancy off once it is on.  
> ---Apple Docs for `NSVisualEffectsView`

Gee, thanks for adding vibrancy to the container view, then!

## Deactivating Vibrancy

Either you don't use `NSSplitViewController` or you deactivate vibrancy in the split view again. I imagine this is wasting CPU cycles, though. 

To deactivate, you have to use your own `NSVisualEffectsView` and set its `state` property to `NSVisualEffectState.inactive`. This also works from within interface builder:

1. Embed the split view item's contents in a `NSVisualEffectsView`;
2. Set or leave "Material" as "Match Appearance" (that is, don't change anything from the next effects view in the hierarchy);
3. Set "Blending Mode" to "Within Window" -- or else the artifacts will stat.
4. Set "State" to "Inactive";
5. Set the view's appearance from "Vibrant Dark" to "Aqua". That's at the bottom of the attributes inspector where nothing interesting happens most of the time. If you miss this step, the table view cells will adjust the cell color to match the supposedly dark background and use a white text color automatically.

## Optional: Fixing Editable Cells not Drawing White Background

Maybe it is just me, but figuring out custom table view backgrounds and adjusting the table cells accordingly is a rather painful experience. Even when the table looks right, editing the table cell contents now didn't display opaque backgrounds.

If you enable "Draw Background" on the table cell's `NSTextField`, you always get a white background.

You have to provide a background color before the window's field editor appears. (Or you have to provide a custom field editor from your `NSWindowController`, but that will make your window drawing super slow.)

My favorite solution so far: program editing a cell myself.

The `NSTableView` behavior is set to "none". I handle double-clicks and the Enter key myself and call `cellView.beginEditing()` on the selection. The cell view is configured thus:

```swift
class ResultTableCellView: NSTableCellView, NSTextFieldDelegate {

    func beginEditing() {

        guard let textField = self.textField else { return }

        textField.isEditable = true
        textField.isSelectable = true

        textField.selectText(nil)

        if let fieldEditor = textField.currentEditor() {

            fieldEditor.drawsBackground = true
            fieldEditor.backgroundColor = NSColor.white
            fieldEditor.textColor = NSColor.black
        }
    }

    private func endEditing() {

        guard let textField = self.textField else { return }

        textField.isEditable = false
        textField.backgroundColor = NSColor.clear
        textField.isSelectable = false

        textField.needsDisplay = true
    }

    override func controlTextDidEndEditing(_ obj: Notification) {
        
        endEditing()
    }
}
```

But that wasn't quite enough. The field editor appearance was still mangled by default. So I had to stop the `NSTextFieldCell` from changing the field editor's configuration:

```swift
class ResultTableTextFieldCell: VerticallyCenteredTextFieldCell {
    override func setUpFieldEditorAttributes(_ textObj: NSText) -> NSText {
        // Doing nothing is necessary to keep the editor white.
        return textObj
    }
}
```

And then, finally!, the text field obtains a white background when editing. When the user is not editing, the underlying `NSTableRowView`'s background can shine through.
