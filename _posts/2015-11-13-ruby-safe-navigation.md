---
title: "Ruby 2.3 Will Have Optional Chaining"
created_at: 2015-11-13 17:47:26 +0100
kind: worklog
tags: [ ruby ]
comments: on
preview: fulltext
---

Swift, C#, and Groovy had it -- now Ruby 2.3 will get it, too: a "safe navigation operator", or what we call "optional chaining".

Instead of:

    #!ruby
    if u && u.profile && u.profile.thumbnails && u.profiles.thumbnails.large

You can now write (with the beta):

    if u&.profile&.thumbnails&.large

They didn't use the `?` because Ruby method names usually have a postfix question mark when they return a boolean; also, they will have a postfix bang when they mutate the receiver. This would've led to confusion.

Since Swift came out, I haven't used Ruby a lot, really. I still use it for all my scripting needs, but coding real applications? Rarely. I really like strong typed languages for that: leaning onto the compiler for refactorings and such.
    
[See the Ruby enhancement issue](https://bugs.ruby-lang.org/issues/11537) if you're curious.

