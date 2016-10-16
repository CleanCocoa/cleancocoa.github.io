---
title: The Surprising Intricacies of Editing Parts of NSTableView
created_at: 2016-05-25 10:51:38 +0200
kind: worklog
tags: [ nstableview, editing, field-editor ]
image: 201605251059_nstableview-components.png
comments: on
---


Editing `NSTableView` cells with a double-click is the default behavior. I wrote about [how to make the column header cells editable][head] already. But in order to figure out how to end an active editing session when the user hits **&#x2318;S** to save the document, I grew more and more disappointed by the day.

On iOS, everything is `UIView`-based. On the Mac, though, you have the legacy of "cells". They are a bit more primitive ancestors to views. They are little boxes that draw stuff and may be editable. They have nothing to do with table cells. Most components we use on the Mac are `NSView`-based, though, a lot of them wrapping ancient `NSCell`s for the action.

{% include figure.md src="/assets/blog/2016/201605251059_nstableview-components.png" alt="diagram of classes involved" caption="AppKit classes you need to take into account to create a table and make it editable, plus a few others for illustration. Table-related classes are highlighted" %}

I put a few common components in a diagram and highlighted the stuff you need to build a `NSTableView` to make sense of all this. You see that the table-related things are scattered all around the place and that `NSTableHeaderCell` is a real oddball.

`<ConfusingExplanation>`

The actual _table_ cells in `NSTableView` are of `NSTableCellView` kind. They contain a `NSTextField` to display text. And as long as they are editable, this very text field turns into a text box on double click. "Turning into a text box" is not very accurate. In fact, the containing `NSWindow` owns a so-called _field editor_. It's a `NSText` object that is hidden -- unless you edit a text field, in which case it takes the text field's content and draws itself over that very text field. It looks as if you move the cursor into the text field, but in reality you make the field editor draw itself above the text field for editing. Crazy. But I guess it was easy on the system's resources back in the day. We utilized this knowledge to make [column header cells appear editable][head] already.

The **widgets** you place in Interface Builder are mostly either dumb `NSView`-based containers to display stuff or `NSControl`-based objects to respond to user interaction. `NSTextField` and `NSButton` are such controls. The table header draws each column's header cells. The header cells aren't `NSView`-backed though. And `NSTableColumn`s are no views at all -- they're more like view models. In short, it's not unified and can cause quite some trouble.

The thing is that `NSTextField` and the field editor's `NSText` (or, more specifically, `NSTextView`) have nothing in common. One uses the other. Also, the `NSTextFieldCell` that backs a column header has nothing in common with `NSTextField`. 

`</ConfusingExplanation>`

Prize question: **How do you end the user's editing session?** When she hits **&#x2318;S**, the text that is currently being edited is not yet part of the underlying widget. It's only visible in the field editor that is drawn above it. So you have to end editing first to commit the changes.

Editing sessions inevitably utilize a field editor, so the only chance you have is to subscribe to notifications. If you have a `NSDocument`-based application, you'll have to filter out the notifications from unrelated documents, though, or else all your documents are going to be synchronized.

In ["How do I detect start and end edit sessions of a cell in `NSTableView`?"](https://developer.apple.com/library/mac/qa/qa1551/_index.html) (QA1551), Apple kindly instructs us to use the appropriate delegate methods. These only work for editing table cells, though. 

* `control:textShouldBeginEditing:` is called for subclasses of `NSControl` that are being edited. But header cells are no subclasses of `NSControl`, so you don't get the same behavior.
* `NSControlTextDidBeginEditingNotification` is posted by `NSControl` subclasses. The field editor is not a subclass of `NSControl`, so you will still only get this notification for table cells.

In terms of delegates, you're left with `NSTextViewDelegate.textShouldBeginEditing(_:)` to notice that the header cell is being edited. The best place to put this is in a subclass of `NSTableHeaderCell`. Just as we did [to make it editable][head] in the first place.

In terms of notifications, `NSTextDidBeginEditingNotification` is fired for both the manual summoning of a field editor in the header cells and for editing table cell text fields.

So there _is_ a universal way to get notified when the user edits text on the level of `NSText`. But it's so different from the delegate methods I have in place that I wonder how I could make use of this without duplicating logic.

[head]: /posts/2016/04/edit-nstableview-header-cell/
