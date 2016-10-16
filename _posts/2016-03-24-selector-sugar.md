---
title: Swift Selector Syntax Sugar
created_at: 2016-03-24 08:06:00 +0100
kind: worklog
tags: [ swift, extension ]
url: https://medium.com/swift-programming/swift-selector-syntax-sugar-81c8a8b10df3#.kw875dmkj
comments: on
---

The gist of the article is that we can omit the verbose `#selector` syntax by adding a convenience property:

    #!swift
    private extension Selector {
        static let buttonTapped = 
            #selector(ViewController.buttonTapped(_:))
    }
    ...
    button.addTarget(self, action: .buttonTapped, 
        forControlEvents: .TouchUpInside)

I'd make that not `private` but `internal` and group these in a struct to separate them from other `Selector` stuff:

    #!swift
    extension Selector {
        // One for each view module ...
        struct Banana {
            static let changeSizeTapped = 
                #selector(BananaViewController.changeSizeTapped(_:))
        }

        struct Potato {
            static let peelSelected = 
                #selector(PotatoViewController.peelSelected(_:))
        }
    }
    
    // ...
    
    button.addTarget(self, action: .Potato.peelSelected, 
        forControlEvents: .TouchUpInside)

Great tip!
