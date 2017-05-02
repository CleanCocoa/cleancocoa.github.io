---
title: Drawing Custom Alternating Row Backgrounds in NSTableViews with Swift
created_at: 2017-05-02 09:15:55 +0200
tags: [ nstableview ]
---

This is an old hat in Cocoa: when you change the appearance of `NSTableRowView`s, they will indeed look different -- but the enclosing table view itself will still draw the system default background to fill the space below the last row.

Similarly, when you have scrolling elasticity enabled and scroll above the topmost row, whatever is being drawn there won't match your custom styled rows, either.

`NSTableView`s themselves draw the row backgrounds that visually match the `NSTableRowView` backgrounds, but only in default settings. When you change row backgrounds, you end up with colorful content rows -- and regular white & light gray rows above and below the content, filling the gap to the edges of the table's clip view.

As I said, this is an old hat. But now we work with Swift and all, so I figured I'd share the Swift code I use to draw custom alternating row backgrounds above and below the visible content:

```swift
public class CustomTableView: NSTableView {
    
    var alternateBackgroundColor: NSColor = // ...
    
    public override func drawBackground(inClipRect clipRect: NSRect) {

        super.drawBackground(inClipRect: clipRect)

        guard usesAlternatingRowBackgroundColors else { return }

        drawTopAlternatingBackground(inClipRect: clipRect)
        drawBottomAlternatingBackground(inClipRect: clipRect)
    }

    fileprivate func drawTopAlternatingBackground(inClipRect clipRect: NSRect) {

        guard clipRect.origin.y < 0 else { return }

        let backgroundColor = self.backgroundColor
        let alternateColor = self.alternateBackgroundColor

        let rectHeight = rowHeight + intercellSpacing.height
        let minY = NSMinY(clipRect)
        var row = 0

        while true {

            if row % 2 == 0 {
                backgroundColor.setFill()
            } else {
                alternateColor.setFill()
            }

            let rowRect = NSRect(
                x: 0,
                y: (rectHeight * CGFloat(row) - rectHeight),
                width: NSMaxX(clipRect),
                height: rectHeight)
            NSRectFill(rowRect)

            if rowRect.origin.y < minY { break }
            
            row -= 1
        }
    }

    fileprivate func drawBottomAlternatingBackground(inClipRect clipRect: NSRect) {

        let backgroundColor = self.backgroundColor
        let alternateColor = self.alternateBackgroundColor

        let rectHeight = rowHeight + intercellSpacing.height
        let maxY = NSMaxY(clipRect)
        var row = rows(in: clipRect).location

        while true {

            if row % 2 == 1 {
                backgroundColor.setFill()
            } else {
                alternateColor.setFill()
            }

            let rowRect = NSRect(
                x: 0,
                y: (rectHeight * CGFloat(row)),
                width: NSMaxX(clipRect),
                height: rectHeight)
            NSRectFill(rowRect)

            if rowRect.origin.y > maxY { break }
            
            row += 1
        }
    }
}
```

I don't attempt to draw row backgrounds in my `NSTableView` here. I leave that to the `NSTableRowView` itself. And its color is set using the `NSTableViewDelegate`.

```swift
func tableView(_ tableView: NSTableView, didAdd rowView: NSTableRowView, forRow row: Int) {
    
    guard let tableView = tableView as? CustomTableView else { return }
    
    rowView.backgroundColor = row % 2 == 1
        ? tableView.backgroundColor
        : tableView.alternateBackgroundColor
}
```

If you don't just want a solid background but draw custom bezeled borders or something even fancier instead, putting everything into the table view drawing code may pay off because you can leave the drawing code in that single place. Your row views will then have to be transparent.
