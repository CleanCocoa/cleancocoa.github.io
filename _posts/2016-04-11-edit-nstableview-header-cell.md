---
title: How to Edit NSTableView Headers with a Double-Click
created_at: 2016-04-11 17:19:40 +0200
kind: worklog
tags: [ nstableview, cocoa, editing, field-editor ]
image: 201604111722_screenshot.png
vgwort: http://vg01.met.vgwort.de/na/e35840c6d78345c3858eb775abc5d858
comments: on
---

When you work with view-based `NSTableView`s, cells by default contain a  `NSTextField`. You can edit cell contents with them. The headers are not designed to be edited, though. You don't have much control over the column headings from Interface Builder. So you have to build this yourself.

I assembled an example project for this for your convenience. [Find it on GitHub.][gh]

{% include figure.md src="/assets/blog/2016/201604111702_tableheader.gif" alt="screencast" caption="The result: it ends editing on enter, tab, and when double-clicking another cell" %}


## Edit on double-click

`NSTableView` conveniently supports setting a `doubleAction` in Interface Builder. From that `@IBAction`, you can find out if the header row was clicked like so:

    #!swift
    @IBAction func tableViewDoubleClick(sender: TableView) {

        let column = sender.clickedColumn
        let row = sender.clickedRow

        // Abort when clicked outside the table bounds
        guard column > -1 else { return }

        if row == -1 {
            editColumnHeader(tableView: sender, column: column)
            return
        }

        editCell(tableView: sender, column: column, row: row)
    }
    
    private func editColumnHeader(tableView tableView: TableView, column: Int) {
        // We'll get to this in a second
    }

    private func editCell(tableView tableView: TableView, column: Int, row: Int) {

        guard row > -1 && column > -1
            let view = tableView.viewAtColumn(column, row: row, makeIfNecessary: true) as? NSTableCellView
            else { return }
            
        view.textField?.selectText(self)
    }

`NSTableCellView`s have text fields and can be edited easily. Header cells don't work this way, so we have to figure that out ourselves.

## Overlaying header cells with the field editor

I didn't know about the notion of a per-window "field editor" before, but this was the key to make editing header cells possible.

In short, each window has a reusable `NSTextView` that is displayed in each and every editable text component. This is called the **field editor**. When you double click a cell in your table, the field editor visually moves inside the cell's boundaries. When you tab your way through the table, the field editor moves around.

This means that not the visible table cells are edited but an overlay text view. When the field editor is shown, the underlying cell doesn't change. It's invisible because the field editor visually hides it behinds itself. If you type something, the field editor takes the text; the cell still doesn't change. Only when the field editor closes will the cell obtain its new value. Orchestrated properly, all of this is invisible to the user.

We will use the field editor to achieve the same effect since the table header view doesn't support all of this convenience itself.

You can request the field editor from your `NSWindow` instance anytime:

    #!swift
    let targetView: NSView = // ...
    let createIfNecessary = true
    let fieldEditor = self.window?.fieldEditor(createIfNecessary, forObject: targetView)

Using the field editor afterwards takes care of setting the frame correctly. An `NSCell` implements `selectWithFrame(_:, inView:, editor:, delegate: , start:, length:)` to configure the field editor you pass as the `editor` argument properly, selecting `length` characters of the cell's text beginning at `start`.

## Making the header cell editable

I encapsulated calling this method in a custom `NSTableHeaderCell` subclass. The header cell becomes the `NSTextViewDelegate`, too, to take the new value when the field editor goes away:

    #!swift
    class TableHeaderCell: NSTableHeaderCell, NSTextViewDelegate {

        func edit(fieldEditor fieldEditor: NSText, frame: NSRect, headerView: NSView) {

            let endOfText = (self.stringValue as NSString).length
            self.highlighted = true
            self.selectWithFrame(frame,
                inView: headerView,
                editor: fieldEditor,
                delegate: self,
                start: endOfText,
                length: 0)

            // Resetting the style of the field editor to match the header
            fieldEditor.backgroundColor = NSColor.whiteColor()
            fieldEditor.drawsBackground = true
        }

        func textDidEndEditing(notification: NSNotification) {

            guard let editor = notification.object as? NSText else { return }

            self.title = editor.string ?? ""
            self.highlighted = false
            self.endEditing(editor)
        }
    }

This setup places the cursor at the end of the text box instead of selecting the existing content. When the field editor resigns first responder, for example because the user hits the enter key, `textDidEndEditing(_:)` is invoked.

The `editColumnHeader` method I left out at the beginning can now be written as follows:

    #!swift
    extension TableWindowController {
        private func editColumnHeader(tableView tableView: NSTableView, column: Int) {

            guard column > -1,
                let tableColumn = tableView.tableColumn(column: column),
                headerView = tableView.headerView as? NSTableHeaderView,
                headerCell = tableColumn.headerCell as? TableHeaderCell,
                fieldEditor = fieldEditor(object: headerView)
                else { return }

            headerCell.edit(
                fieldEditor: fieldEditor,
                frame: headerView.headerRectOfColumn(column),
                headerView: headerView)
        }
    }    

## Adopt the value the user enters

This will work as expected and set the `TableHeaderCell`'s text to the field editor's value if you hit enter or tab. 

But there's [a common problem](http://stackoverflow.com/a/17481500/1460929): the header doesn't obtain the changed text if you double-click out of the field editor to edit another column heading or a cell. **That's because the field editor doesn't lose focus in these cases and doesn't fire `textDidEndEditing`.** The field editor is never destroyed; and when you edit another cell, you merely change its contents and draw it in another place.

There are various suggestions about how to trigger the `textDidEndEditing` event nevertheless. I found a very easy trigger: when a new field editor is requested. That's exactly when the existing field editor may change positions.

First, support a manual reset in a custom field editor:

    #!swift
    class HeaderFieldEditor: NSTextView {

        static let ManualEndEditing = "field editor will change"

        func switchEditingTarget() {

            guard let delegate = self.delegate else { return }

            let notification = NSNotification(name: HeaderFieldEditor.ManualEndEditing, object: self)
            delegate.textDidEndEditing?(notification)
        }
    }

I could (or should?) fire a proper `NSTextDidEndEditingNotification` to indicate the text field loses focus, too, but I don't need that in my app (I'm not observing anything on my own and rely on the delegate method) and I worry about unintended side-effects. Keep in mind that `textDidEndEditing` is only a convenience mechanism we exploit here.

We can call this method when a field editor is requested. The window has a default field editor. To use our own, we can use its `NSWindowDelegate`'s `windowWillReturnFieldEditor(_:, toObject:)` method and return a custom object instead.

Since a field editor is requested _for every cell_ when the table is displayed, we will limit the use of the custom field editor to table header views:

    #!swift
    class TableWindowController: NSWindowContoller {
        // ...
        
        lazy var headerFieldEditor: HeaderFieldEditor = {
            let editor = HeaderFieldEditor()
            editor.fieldEditor = true
            return editor
        }()
    }
    
    extension TableWindowController: NSWindowDelegate {
        
        func windowWillReturnFieldEditor(sender: NSWindow, toObject client: AnyObject?) -> AnyObject? {

            // Return default field editor for everything not in the header.
            guard client is TableHeaderView else { return nil }

            headerFieldEditor.switchEditingTarget()

            return headerFieldEditor
        }
    }

This in combination with the regular `textDidEndEditing` event suffices.

## Making the result pretty

The field editor will be too high by default. I tried a few values and came up with a hacky but working solution, hard-coding the frame change values in a header view subclass. I found 4pt vertical padding to do the trick and show the field editor right above the header cell:

    #!swift
    class TableHeaderView: NSTableHeaderView {

        /// Trial and error result of the text frame that fits.
        struct Padding {
            static let Vertical: CGFloat = 4
            static let Right: CGFloat = 1
        }

        /// By default, the field editor will be very high and thus look weird.
        /// This scales the header rect down a bit so the field editor is put
        /// truly in place.
        func paddedHeaderRect(column column: Int) -> NSRect {

            let paddedVertical = CGRectInset(self.headerRectOfColumn(column), 0, Padding.Vertical)
            let paddedRight = CGRect(
                origin: paddedVertical.origin,
                size: CGSize(width: paddedVertical.width - Padding.Right, height: paddedVertical.height))

            return paddedRight
        }
    }

Using this in the double click handler:

    #!swift
    private func editColumnHeader(tableView tableView: NSTableView, column: Int) {

        guard column > -1,
            let tableColumn = tableView.tableColumn(column: column),
            headerView = tableView.headerView as? TableHeaderView,
            headerCell = tableColumn.headerCell as? TableHeaderCell,
            fieldEditor = fieldEditor(object: headerView)
            else { return }

        headerCell.edit(
            fieldEditor: fieldEditor,
            frame: headerView.paddedHeaderRect(column: column),
            headerView: headerView)
    }

## Conclusion

There's one caveat I didn't yet solve: when `switchEditingTarget` is called, during `textDidEndEditing`, the header cell will call `endEditing(_:)` to end the editing session -- thereby triggering a `NSTextDidEndEditingNotification` which calls the `textDidEndEditing` method again.

**Update**: Found something. Just call `endEditing` manually:

    #!swift
    class HeaderFieldEditor: NSTextView {

        func switchEditingTarget() {

            guard let cell = self.delegate as? NSCell else { return }

            cell.endEditing(self)
        }
    }

That solves the issue.

Another thing that's missing in the sample: tabbing through the header cells. You can double-click to edit them, but hitting tab will only end the editing session. I suppose you have to fire a different event when tab is pressed and select the next header from the view controller.

[Look at the sample project on GitHub][gh] and play with it yourself.

[gh]: https://github.com/DivineDominion/Editable-NSTableView-Header
