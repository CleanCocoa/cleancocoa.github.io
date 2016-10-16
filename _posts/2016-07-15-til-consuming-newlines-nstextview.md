---
title: "TIL: How to Make NSTextView Reject Newlines"
created_at: 2016-07-15 13:32:55 +0200
kind: worklog
tags: [ appkit, til, pasteboard ]
comments: on
---

Today I learned that a `NSWindow`'s field editor cannot trivially be set to "single line mode." A field editor is a `NSText` subclass like `NSTextView`, but `NSTextField` is only a configurable widget that _uses_ the field editor. 

How does the "single line mode" setting do what it does? I have no clue.

So how do you make custom field editor classes reject newlines both through pressing return, alt-return, and pasting text with newline characters? You override its `NSResponder` hooks!

    #!swift
    class CustomFieldEditor: NSTextView {

        // ... imagine some relevant customizations here ...
        
        override func insertNewline(sender: AnyObject?) {

            // consume and discard
        }

        override func insertNewlineIgnoringFieldEditor(sender: AnyObject?) {

            // consume and discard
        }

        override func paste(sender: AnyObject?) {

            let pasteboard = NSPasteboard.generalPasteboard()

            guard let pasteboardItem = pasteboard.pasteboardItems?.first,
                text = pasteboardItem.stringForType(ContentTypes.text)
                else { return }

            let cleanText = text.stringByReplacingOccurrencesOfString("\n", withString: " ")

            self.insertText(cleanText)
        }
    }
    
    enum ContentTypes {
        static let text = "public.utf8-plain-text"
    }
