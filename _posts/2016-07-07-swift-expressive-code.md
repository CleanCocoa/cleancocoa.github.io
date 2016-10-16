---
title: I love Swift's Expressiveness
created_at: 2016-07-07 09:59:07 +0200
kind: worklog
tags: [ swift, enum ]
vgwort: http://vg01.met.vgwort.de/na/e100dc6dc87d4bfcb66f2480e15b7d2a
comments: on
---


Take optionals, for example. Optionals demarcate when something's not there at all.

    #!swift
    let result: Int? = ...

When an optional is `nil`, this may signify something went wrong. That's much more explicit than relying on a convention like returning `-1` for failed requests. 

This is convenient and changes the game of handling edge cases completely. Because whenever you receive an optional, you have to deal with its two-fold nature. Checking for `-1` can slip your mind. Forgetting to unwrap an optional is unlikely.

The classic `NSNotFound` convention can (and, as I argue, _should_) be replaced by optionals instead. That's one convention less to have in your mind. One liability less. Instead of relying on knowledge on the programmer's side, put the facts into the code.

Taking it a step further, network requests may be better off being represented as a custom enum instead of an optional. Because then you can give the results a real name. The `Optional` type is a generic enum with the `.none` and `.some` case -- names that indicate presence of a value. But the mental mapping isn't very good for failure indication. Instead, cook your own. It's cheap:

    #!swift
    enum RequestResult<T> {
        case .loading
        case .failure
        case .success(T)
    }
    
    var result: RequestResult<Int> = ...

People praise Swift's expressiveness which is a magnitude higher than Objective-C's. I think they have a point. It's these things in my day to day work that put a smile on my face. I'm happy when I come up with stuff like that. Sometimes I look at conditions and ask myself if I can cast them in stone, so to speak. To encapsulate the knowledge of branching-off or checking values for validity -- to make a type from it.

Whenever I think about representing a failed request of `[Something]` as an empty array to avoid branching in code, I pause and step back. Sure this is convenient. But will the call site need to know about the difference between success and failure? I can always map a failure to an empty collection on the call site later. But not having knowledge about how things went in the first place? That's not very useful when it comes to network requests. (But I care less in case of some local operations.) So I may add an enum to express when I need to consider branching in code at the call site.
