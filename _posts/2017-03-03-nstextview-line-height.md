---
title: Setting the Line Height of a NSTextView
created_at: 2017-03-03 09:42:16 +0100
kind: worklog
tags: [ typesetting, typography, editor ]
comments: on
---

`NSTextView` (and `UITextView` for that matter) have a `defaultParagraphStyle` attribute where you can set the text's line height. That works swell -- if you display text statically. Once the user can enter something, you can run into trouble:

* If the user adds text in the middle of an already laid-out line of text, the paragraph style is retained. 
* If the user writes at the beginning of the line, the line height info gets lost.

{% include figure.md src="/assets/blog/2017/20170303102946_losing-paragraph-info.gif" alt="GIF of the process" caption="This is what happens when you type at the beginning of a line" %}

It's your usual RTF nightmare. I know this behavior from rich text editors; and I developed my own way to make sense of it in the process. It might not be what is really going on, but it's a good heuristic: it's just like the opposite of making a word bold, placing your cursor after that word, type, and get more bold text. There, the "bold text" information is carried on. The cursor inherits this info from the character left to it. But if you start at the beginning of a line, your cursor will not inherit what comes afterward. And since there is nothing before its position, it starts with empty info, and thus empty line height settings. Since the whole paragraph is affected by this, the latest change wins. Beginning to type at the beginning of a paragraph with empty paragraph settings removes them from what comes afterwards.

So this might not be The Truth, but it helps me deal with shitty software. I don't want to write shitty software, though, so I look for ways out of this. I don't intend the user to change paragraph settings; I want the text view to have a certain look and feel no matter what gets pasted or typed in it.

Hunting for Core Text/TextKit callbacks, `NSTextStorageDelegate` seems to provide a good customization point:

    #!swift
    func textStorage(
        _ textStorage: NSTextStorage, 
        didProcessEditing editedMask: NSTextStorageEditActions, 
        range editedRange: NSRange, 
        changeInLength delta: Int
    ) {
        let paragraphStyle = NSMutableParagraphStyle()
        paragraphStyle.lineHeightMultiple = 2.0
        textStorage.addAttributes([NSParagraphStyleAttributeName : paragraphStyle], range: editedRange)
    }

Of course it makes sense to store the global `paragraphStyle` once and re-apply it here. I don't know if this is the best place to put it, though. Re-applying all the `NSAttributedString` settings _while typing_ might not perform best.

Also, this is affecting the "rich text representation" of the text. If you copy the result and paste it into TextEdit, say, the text will look the same, line height settings and all.

You can override the pasteboard representation to be "plain text" only in order to remove the style info and thus have it behave like the "Paste and Match Style" command from the "Edit" menu automatically.

Again, I don't know if this is the best way to do this.

What I'd expect to create instead:

* a text view with a "plain text" representation by default (the model)
* typesetting customizations that only affect what is visible on screen (the view)

I imagine this to be like HTML code/browser rendering, not like <abbr title="What You See Is What You Get">WYSIWYG</abbr>. What you type is not what you see. Just what you'd expect a source code editor to be like.

I'll keep you posed as I dive deeper into TextKit and stuff.
