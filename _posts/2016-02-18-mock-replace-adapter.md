---
title: "How do You Really Mock Objects You Don't Own? You Replace Them with Adapters"
created_at: 2016-02-18 14:34:37 +0100
kind: worklog
tags: [ testing, mocking, adapter, protocol ]
shared_image: "blog/201506031611_strawman-t.jpg"
vgwort: http://vg01.met.vgwort.de/na/41cf2e231b554cba8504385ff2d501c5
comments: on
---

How do you test `NSURLSession` properly? 

After all, using it in tests directly will hit the servers; that's not what you want to do 1000s of times per day. Even if you use a `localhost` based server replacement the tests will be slower and potentially error-prone as the server replacement may not be 100% accurate. Anyway -- there's [a popular series][post] of posts about that topic. There, you'll learn how to mock `NSURLSession` in your tests. I think the advice is good to move on quickly, but it only shifts the problem slightly out of sight. There are better alternatives.


## Inject a Protocol You _Do_ Own

{% include figure.md src="/assets/blog/blog/201506031611_strawman.jpg" alt="strawman" caption="Photo Credit: <a href=\"https://www.flickr.com/photos/earl258/2448032465/\">稻田 ricefilm Strawman</a> by <a href=\"https://www.flickr.com/photos/earl258/\">earl258</a>. License: <a href=\"https://creativecommons.org/licenses/by-nc/2.0/\">CC-BY-NC 2.0</a>" %}

The rationale is simple: you should not provide test doubles for things you don't own. So in [the first part][post] of the series, Joe points out what you need to do instead:

> 1. `HTTPClient` needs to work with an `NSURLSession`
> 2. `NSURLSession` needs to conform to a protocol so we can mock it under test
> 3. We need to mock `NSURLSessionDataTask` so we can assert `resume()` is called

The approach is very clever: instead of working with `NSURLSession` you create a shadow protocol. It consists of methods `NSURLSession` implements already. Then you make `NSURLSession` conform to the protocol without any extra work to implement actual functionality.

The protocol looks like this:

    #!swift
    typealias DataTaskResult = (NSData?, NSURLResponse?, NSError?) -> Void
    
    protocol URLSessionProtocol {
        func dataTaskWithURL(url: NSURL, completionHandler: DataTaskResult)
          -> NSURLSessionDataTask
    }
    
Here's the magic:

    #!swift
    extension NSURLSession: URLSessionProtocol { }

That's it. Now you can pass `NSURLSession`s wherever your app expects an `URLSessionProtocol`. Your app doesn't directly depend on `NSURLSession` anymore!

Conceptually, you've decoupled your code from Apple's framework. This is important and makes testing easy: your test doubles only have to implement the protocol *without* having to do anything with `NSURLSession`. This is huge.

But the problem is this: you changed the label and narrowed down the interface from the plethora of methods `NSURLSession` offers to just one. But your protocol depends on `NSURLSession`'s interface to not change. If its method signature changes ([as is to be expected with Swift 3][swift3]) you have to change the protocol, too.

So your code has a reason to change that's not up to you. In other words it's still coupled to `NSURLSession`, only the dependency is not transparent anymore.

## Decouple the App from the Framework Using Adapters

You don't in fact have achieved anything except making tests a bit more robust: if you subclass `NSURLSession`, some method calls could still hit the network unless you override all of them. The protocol alleviates all that pain.

The real cure against strong coupling to classes you don't control is to push the framework further to the boundaries of your app. In practice, that is quite simple: create your own wrapper. You can even keep the protocol if you like.

If you name your protocols properly to get stuff under test quickly at first (because tests are better than no tests), you don't even have to change names when you later create a wrapper. "`URLSession`" could be protocol, struct, or class.

Wrappers or adapters are very simple to create. Here's a first attempt:

    #!swift
    typealias DataTaskResult = (NSData?, NSURLResponse?, NSError?) -> Void
    
    class URLSession {
        let session: NSURLSession

        init(_ session: NSURLSession) {
            self.session = session
        }
    
        func dataTask(url url: NSURL, completionHandler: DataTaskResult) -> NSURLSessionDataTask {
            
            return session.dataTaskWithURL(url, completionHandler: completionHandler)
        }
    }

This is my own type. **If the `NSURLSession` changes its method signature, I only have a single point of change.**

But the new type still mimics Apple's class. Let's say I want to obtain data from URLs more conveniently throughout my app and don't reuse sessions after all. Then I can make both part of the constructor and simply fire off the task with this slightly modified object:

    #!swift
    class URLDataTask {
        let session: NSURLSession
        let url: NSURL

        init(session: NSURLSession, url: NSURL) {
            self.session = session
            self.url = url
        }
    
        func dataTask(completionHandler: DataTaskResult) -> NSURLSessionDataTask {
            
            return session.dataTaskWithURL(url, completionHandler: completionHandler)
        }
    }

If you decorate Cocoa classes with your protocols, you only achieve a very shallow decoupling. These custom types are as good as it gets.You could even replace the mechanism to fetch data with `AFNetworking` or your own stuff here -- the rest of the app won't notice at all.

- High testability ✔️
- Decoupling of your app from the framework ✔️
- Types that play according to *your* rules ✔️

Please keep in mind, though, that Joe's approach to use protocols to pseudo-extend Cocoa classes to make them your own is great and accomplishes most of the goals, except decoupling. This technique should be in your tool-belt, too.

[swift3]: https://www.reddit.com/r/swift/comments/43a3ib/chris_lattner_swift_2_to_swift_3_will_be_a/
[post]: http://masilotti.com/testing-nsurlsession-input/
