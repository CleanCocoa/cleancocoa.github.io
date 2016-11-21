---
title: Exploring Mac App Development Strategies, 4th Edition
layout: page
---

<script
    id="fsc-api"
    src="https://d1f8f9xcsvx3ha.cloudfront.net/sbl/0.7.1/fastspring-builder.min.js"
    type="text/javascript"
    data-storefront="cleancocoa.test.onfastspring.com/popup-cleancocoa"
    data-debug="true">
</script>

<div class="hero">
    <div class="hero__cover">
        <a href="" class="imagelink"><img src="/assets/books/mac-app-dev_swift3.jpg" alt="book cover" title="Exploring Mac App Development Strategies" class="hero__image"/></a>
    </div>

    <h2 class="hero__heading">Don&rsquo;t worry about big software design decisions anymore</h2>
    
    <p class="hero__text">
        Driven by example, this book applies useful patterns of clean software architecture to macOS app development. Take on app development with solid principles, guided by extensive examples and explanation.
    </p>

    <div class="actions">
        <ul class="actions__buying">
            <li class="actions__buying__item">
                <a class="action action--buy action--buy--main" href="https://cleancocoa.onfastspring.com/exploring-mac-app-development-strategies" data-fsc-action="Reset,Add,Checkout" data-fsc-item-path-value="exploring-mac-app-development-strategies"><span class="cta">Buy for $12</span><br>(PDF, EPUB, & Kindle)</a>                
            </li>

            <li class="actions__buying__item">
                <a class="action action--aux" href="https://geo.itunes.apple.com/us/book/exploring-mac-app-development/id1026837791?mt=11&at=11lxCd">Buy on iBooks Store</a>
            </li>
        </ul>
    
        <div class="actions__misc">
            <a class="action action--download" href="http://cleancocoa.s3.amazonaws.com/develop-mac-apps-clean-architecture-swift-sample.pdf">Free Sample</a>
        </div>
    </div>
    
    <div class="smallprint">
        <ul>
            <li>Lifetime access to updates</li>
            <li>100% Money Back Guarantee: Because I know you’ll love it!</li>
        </ul>
    </div>

    <hr class="hero__rule"/>
</div>

Gain confidence that your stuff is really working. Even if you don't do it consciously, you apply structure your code. Take ownership of the process by understanding how to employ basic software architecture to your advantage. Know your options and design your code according to insight, not tradition.

Learn how to ...

* make your code base robust,
* design reusable code,
* get a solid test harness up and running, and
* properly separate the components of your application.

All this will make your application easier to change and easier to maintain. It will make you a happier developer, even when you're out exploring new parts of the Cocoa Framework.

To develop Mac applications, you'll need to know why and how to separate components. Using the latest technology and a head-start into Swift programming, this book will show you the pro's and con's of several design decisions and discuss alternative approaches so you get to know your options.

## Core Ideas of the Book

We'll explore useful patterns to make the code manageable. Here are some of the topics I explore, discuss, and challenge in this book:

1. Apple's **Core Data is a very invasive framework.** It works best when you couple everything, beginning at the user interface, to Core Data entities. Cocoa Bindings make a lot of the magic possible, but they come at a cost: the Core Data entities (or `NSManagedObject` subclasses) are bound to do a lot of different things for different clients, whereas by 'client' I mean: view components, background services, and the like. It'd be easier to maintain the code if you model entities for each "[Bounded Context](http://martinfowler.com/bliki/BoundedContext.html)" separately.
2. Adopting Domain-Driven Design thinking, **small service objects are favorable**. It's easy to create Data Transfer Objects in Swift using `struct`s, for example. Core Data should be pushed from the Domain, where the business logic is hosted, to an outer layer of the application and be degraded to a mere implementation detail.
3. Once you free your app from the entanglement by Core Data (and other Apple suggested best practices which don't scale well), you will be able to leverage powerful design patterns. I created domain event wrappers around `NSNotification` to **make event handling easier and failure-proof**. Using events to propagate changes (instead of using Bindings, for example) makes concurrent programming easier.

That's part of the topics we cover in this book. You won't be a software architecture expert afterwards, but you'll know how to make your code better, designed to help you prevent bugs

## Content Outline

* Introduction
    * Patterns and principles
    * Architecture overview
* Part 1: Bootstrapping
    * Getting comfortable with Swift, porting Objective-C code to recreate the problem space
    * Get Core Data and related tests running
    * Prepare the user interface in its own layer
    * Have tests in place for the app's domain
* Part 2: Sending Messages Inside of the App
    * Naive event handling
    * Ports &amp; Adapters-approach
* Part 3: Putting a Domain in Place
    * Sending Domain Events
    * Handling Errors
    * Doing real work in the background
* Part 4: Using Core Data for Convenience
    * Undoing the architectural separations
    * Reading it backwards: how to refactor from Core Data to Domain Model
* Epilogue
* Appendix
    * Interesting links
    * On Swift
    * About Objective-C API compliance-related problems

## Contributing

I host the book manuscript on GitHub and distribute it under a [Creative Commons Attribution-ShareAlike 4.0](http://creativecommons.org/licenses/by-sa/4.0/) license. That means you can edit and re-use parts of it. Pull requests are always welcome!

Feel free to contribute to the book and the project on GitHub:

* [Book Manuscript on GitHub][1]
* [Project Code on GitHub][2]

[1]: https://github.com/CleanCocoa/mac-appdev-book
[2]: https://github.com/CleanCocoa/mac-appdev-code



<div class="actions">
    <ul class="actions__buying">
        <li class="actions__buying__item">
            <a class="action action--buy action--buy--main" href="https://cleancocoa.onfastspring.com/exploring-mac-app-development-strategies" data-fsc-action="Reset,Add,Checkout" data-fsc-item-path-value="exploring-mac-app-development-strategies"><span class="cta">Buy for $12</span><br>(PDF, EPUB, & Kindle)</a>                
        </li>

        <li class="actions__buying__item">
            <a class="action action--aux" href="https://geo.itunes.apple.com/us/book/exploring-mac-app-development/id1026837791?mt=11&at=11lxCd">Buy on iBooks Store</a>
        </li>
    </ul>

    <div class="actions__misc">
        <a class="action action--download" href="http://cleancocoa.s3.amazonaws.com/develop-mac-apps-clean-architecture-swift-sample.pdf">Free Sample</a>
    </div>
</div>


## FAQ

### Q: It's written in Swift! What about the frequent Swift language updates?

I'll keep the book and the code up to date so you are always equipped with the latest technology. Apart from updates depending on Swift, I'll update contents based on reader feedback and my own experience. 

I intend this book to be an investment in your indie career. It will accompany your journey until the world's end -- or until Macs stop to work the way they do.

### Q: How can I get the updated version?

Just use the download link you received after your order! It'll work for every future update and is bound to your order number. Just shoot me an e-mail if you lost it and I'll give you a new one.

### Q: Do you offer a money-back guarantee?

Totally. Either you're 100% happy or you get your money back, guaranteed for a lifetime. -- Mine, first and foremost, because nobody else checks my mail :)

