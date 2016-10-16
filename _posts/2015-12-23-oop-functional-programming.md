---
title: Encapsulate a Process in a Single Line Using Bind and Good OO Design
created_at: 2015-12-23 15:41:38 +0100
kind: worklog
tags: [ oop, functional-programming ]
comments: on
---

Watch Saul Mora's AltConf presentation ["Object-Oriented Functional Programming: The Best of Both Worlds!"][conf] to learn more about real world use of `bind` and other functional concepts in object-oriented programming. His examples are really good. 

His use of `bind()` to chain calls which return optionals is even better with a custom operator as syntactic sugar: `>>=`. Then you can see the whole process with meaningfully named action steps in a single line.

I'm not a fan of custom operators per se, but I see their use as long as they adhere to some kind of convention. If they don't, they make people lives too hard too easily.

Saul's example contains a set of functions or closures that take an optional and transform it into another optional to find out if a file as some URL has expired. I stripped the implementation of the steps to stress the chaining:

    #!swift
    func expired(fileURL: NSURL) -> Bool {
        let fileManager = NSFileManager()
        var error: NSError?
	
        let filePath: String? = fileURL.path
        let fileExists: (String) -> (String?) = // ...
        let retrieveFileAttributes: (String) -> ([NSObject: AnyObject]?) = // ...
        let extractCreationDate: ([NSObject:AnyObject]) -> NSDate? = // ...
        let checkExpired: NSDate -> Bool? = // ...

        return filePath 
            >>= fileExists 
            >>= retrieveFileAttributes 
            >>= extractCreationDate 
            >>= checkExpired ?? false
    }
    
Compare that to the "before" code:

    #!swift
    let fileManager = NSFileManager()
    if let filePath = fileURL.path {
        if fileManager.fileExistsAtPath(filePath) {
            var error: NSError?
            let fileAttributes = fileManager.attributesOfItemAtPath(filePath, error: &error)
            if let fileAttributes = fileAttributes {
                if let creationDate = fileAttributes[NSFileModificationDate] as? NSDate {
                    return creationDate.isBefore(NSDate.oneDayago())
                }
            }
            else {
                NSLog("No file attributes \(filePath)")
            }
        }
    }

Using `bind` to chain operations is compatible with "Tell, Don' Ask". Even though chaining requires the operations to return a value, reasoning about the chain as a whole is a lot easier. The process fits into a single line. Imagine how splitting up action steps into chained helper objects would look: you'd be jumping between source files forever. 

Have both in your toolkit to respond to new challenges appropriately.

[conf]: https://realm.io/news/altconf-saul-mora-object-orientated-functional-programming/
