---
title: What I Did When I Found Emojis in Strings Always Lost the Last Byte
created_at: 2016-05-03 18:10:53 +0200
kind: worklog
tags: [ emoji, encoding ]
image: 201605031815_hmm.png
comments: on
preview: fulltext
---


Took me a while to not only figure it out but actually _fix_ that Emojis are encoded in a special way using UTF-8.

If things are well on your side, you'll see a "thinking" emoji here: 🤔

I wanted to be able to enter these in text files, read them in, display them, and save the contents in a way that they are visible in the text file again. Sounds trivial, but [every](http://stackoverflow.com/a/23671591/1460929) [double-encoding trick](http://iossolves.blogspot.de/2014/01/encoding-emoji-characters-iosobjectivec.html) I [found](http://stackoverflow.com/a/10981278/1460929) on the web resulted in escaping the unicode char codes (`\ud83e` etc.)

Reading stuff in worked with basic `NSUTF8StringEncoding`, but writing not so much. Turns out the last byte of a line was stripped for some reasen. Turns out all of this was my fault. Thanks to playgrounds for giving me feedback about [matching encoded data](http://stackoverflow.com/a/24254240/1460929).

Emoji seem to take 7 bytes when you look at the UTF8-encoded `NSData` representation. In my tests, it seemed that when an emoji was the last character in a line things went broke. I _had_ to enter characters after it to work. These characters wouldn't be stored, but at least the emoji would be visible.

My fault was that I was padding strings to have a certain length like so:

    #!swift
    extension String {

        func padded(length length: Int, filler: String = " ") -> String {

            return self.stringByPaddingToLength(length, withString: filler, startingAtIndex: 0)
        }
    }

I was not using `NSString` but native Swift `String`s, so I expected things to go well. But they didn't. I don't know for sure why this method from foundation wreaks havoc with strings containing emoji, but it sure did. At first, the fact that 1 emoji character consists of 2 UTF8-encoded unicode values at once resultet in truncating the line. Even when I increased the padding to 2x the visible character count, the last visible character was lost in the end.

I found [a cheat](https://www.objc.io/issues/9-strings/unicode/#length): calculate the byte size of the given string and fill up manually:

    #!swift
    extension String {

        func padded(length length: Int, filler: String = " ") -> String {

            let byteLength = self.lengthOfBytesUsingEncoding(NSUTF32StringEncoding) / 4
            let paddingLength = length - byteLength
            let padding = String(count: paddingLength, repeatedValue: Character(" "))
            return "\(self)\(padding)"
        }
    }

Works like a charm and doesn't use data. [TableFlip](http://tableflipapp.com) beta testers will probably find more edge cases, but at least the app doesn't crash or ruin your files.
