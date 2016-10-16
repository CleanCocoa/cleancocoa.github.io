---
title: Get Your App Indexed by Google
created_at: 2016-01-04 14:20:13 +0100
kind: worklog
tags: [ universal-links, google, seo ]
url: https://possiblemobile.com/2015/12/get-your-app-content-indexed-in-google-search/
comments: on
---

I did know that universal links help integrating web content and native app scenes: regular web links may show the content in the browser on most devices but magically open the content within your native app when it's installed. The prime example I knew was IMDB's movie database, but it obviously works for recipes as well.

Now I [read about a Google App Indexing SDK](https://possiblemobile.com/2015/12/get-your-app-content-indexed-in-google-search/). Before universal links, it tried to achieve the same effect. But what does it do nowadays?

The source code isn't prominently linked anywhere. I went to [the CocoaPods spec](https://github.com/CocoaPods/Specs/blob/5e07088fac755758513e8533b934b779361fc3b1/Specs/GoogleAppIndexing/2.0.2/GoogleAppIndexing.podspec.json) and downloaded the source tar file (at the bottom of the spec).

The "source" contains a framework which publishes two classes, `GSDAppIndexing` and `GSDDeepLink`. There's a [sample iOS app](https://github.com/googlesamples/google-services/tree/master/ios/app-indexing/AppIndexingExampleSwift) on GitHub by Google as well.

You use `GSDAppIndexing` to register your app with its iTunes ID -- but the sample application doesn't do that at all. [The docs](https://developers.google.com/app-indexing/ios/server) indicate this is needed to expose content to crawlers, though. The sample app _does_ handle links in `AppDelegate`:

    #!swift
    var currentDeepLink = String()
    
    func application(app: UIApplication, openURL url: NSURL, options: [String : AnyObject]) -> Bool {
        let sanitizedURL = GSDDeepLink.handleDeepLink(url)
        currentDeepLink = sanitizedURL.absoluteString
        return true
    }

Instead of routing, it only stores the link for display purposes. There you see what `GSDDeepLink` is good for: extracting the real deep links from the Google search deep link call.

What's it good for?

It will show deep links (perhaps more prominently?) when you browse Google on Mobile Safari. In other words, regular visitors with a Mac probably won't notice any difference.

The following is the expected URL scheme according to the `GSDDeepLink.h` file, split up into multiple lines for reading convenience:
    
    gsd-<scheme>://<appstore-id>/
        ?google-deep-link=<url-encoded-original-deeplink>
        &google-callback-url=<url-encoded-callback-url>
        &google-min-sdk-version=<minimum-sdk-version>

And an example:

     Annotation:
     ios-app://544007664/vnd.youtube/www.youtube.com/watch?v=aISUYHTkTOU
     
     Original Deeplink:
     vnd.youtube://www.youtube.com/watch?v=aISUYHTkTOU

     Callback URL:
     googleapp://
     
     Final URL:
     gsd-vnd.youtube://544007664/
         ?google-deep-link=vnd.youtube%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DaISUYHTkTOU
         &google-callback-url=googleapp%3A%2F%2F
         &google-min-sdk-version=1.0.0

The following is just totally made up: I assume the `GSDDeepLink` helper actually sends back some statistics to Google. Otherwise I wouldn't know what the wrapped URL would be good for at all. (Suggestions welcome.) `GSDDeepLink` does overlay the status bar for a short while according to the header documentation so users may return to the search results.

I still don't get the benefit from using Google's framework in my projects. When users reach my website via Google, the native app is expected to open automatically anyway. So there's only the formatting of app-related search results left as a benefit as [screenshots](http://9to5mac.com/2015/10/09/googles-app-indexing-links-coming-to-safari-on-ios-by-end-of-month/) indicate.
