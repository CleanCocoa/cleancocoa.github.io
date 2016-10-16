---
title: Transparent NSTableView Headers
created_at: 2016-08-03 08:57:34 +0200
kind: worklog
tags: [ nstableview ]
image: before-after.png
comments: on
---

An `NSTableView` is usually embedded in a `NSScrollView`. If you set the columns not to span the full width, the two of them will take care of drawing an empty table header cell to the right so that the interface doesn't look weird.

But I want to have a weird interface. Because TableFlip's tables are different than the usual `NTableView` use cases.

{% include figure.md src="/assets/blog/2016/before-after.png" alt="screenshots of before and after the change" caption="Before, the header spans the whole width. After, it visually stops where the table contents stop." %}

Now it should be even easier to see that the header cells are still part of the table and not just a block of color in the interface.

The first step is to make your `NSTableHeaderView` not draw full width but only existing column headers:
    
    #!swift
    class NarrowTableHeaderView: NSTableHeaderView {

        override func drawRect(dirtyRect: NSRect) {

            guard let columns = tableView?.numberOfColumns
                else { return }

            (0...columns)
                .map { headerRectOfColumn($0) }
                .forEach { super.drawRect($0) }
        }
    }

I am using a `NSVisualEffectView` on the window so this looks odd. If you don't use a blur effect, the background color will match the default window's color and you'll be done.

{% include figure.md src="/assets/blog/2016/background-clip.png" alt="screenshot of clip view drawing its own background" caption="The header row draws a gray background on its own" %}

You'll notice that the content section of `NSScrollView` doesn't draw a background while the header section draws the default background to the right of the header. That's because `NSScrollView` uses _two_ `NSClipView`s for what it does. One is the `contentView` and it behaves properly. The other is privately used for the header section and ignores the `NSVisualEffectView` setting. Probably because it is commonly used to draw something anyway.

To make the header clip view look truly transparent in a setup like mine, you have to add another `NSVisualEffectView` to the:

    #!swift
    class TransparentHeaderScrollView: NSScrollView {

        override func awakeFromNib() {

            super.awakeFromNib()

            transparentizeHeaderClipView()
        }

        private func transparentizeHeaderClipView() {

            let clips = self.subviews.flatMap { $0 as? NSClipView }
            guard let headclip = clips.filter({ $0 !== self.contentView }).first,
                content = headclip.documentView as? NSView
                else { return }

            let visualEffectView = NSVisualEffectView(frame: NSRect.zero)
            visualEffectView.material = NSVisualEffectMaterial.Light
            visualEffectView.blendingMode = NSVisualEffectBlendingMode.BehindWindow
            visualEffectView.state = NSVisualEffectState.Active

            headclip.documentView = visualEffectView
            visualEffectView.addSubview(content)
        }
    }

That does the trick.
