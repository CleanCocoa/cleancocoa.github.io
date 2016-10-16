---
title: Don't Build on El Capitan Without Checking App Transport Security
created_at: 2015-10-28 08:33:09 +0100
kind: worklog
tags: [ xcode, mac, ios, ssl ]
comments: on
---

I got burned this week. Pretty bad.

I shipped a small bugfix release for my Mac app _Word Counter_ a couple of days ago to prepare for the "big one" coming this week. Naturally, I built that version on my El Capitan dev machine. I pushed the update to my server. Updates using Sparkle worked. -- But now users of that version cannot *ever* update to the next version. Because I haven't thought about ATS.

ATS stand for [Application Transport Security](https://developer.apple.com/library/prerelease/mac/releasenotes/Foundation/RN-Foundation/) and it essentially entails that any request to non-HTTPS URLs has to be specifically allowed through [security exceptions](http://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/) in your app's Info.plist. Lots of folks recommend to not add exceptions. 

> Application Transport Security is a new security feature whose goal is to protect customer data. If your app is linked on or after OS X 10.11 or iOS 9, all NSURLSession and NSURLConnection based cleartext HTTP loads (http://) will be denied by default. Encrypted HTTPS (https://) connections will also require "best practice" TLS behaviors, such as TLS version and cipher suite requirements. Temporary exceptions can be configured via your app's Info.plist file. For more information, see the WWDC sessions covering Application Transport Security.

Guess who didn't think about changing the Sparkle update feed URL to `https://`. Me.

Updating from older builds works just fine. It's just v1.2.2 that's broken now and possibly forever. Because there's no way I can change the URL post-installation on user's machines.

I'm glad this was just an "Edge" build, affecting all early adopters, but not all users. So a mail to all my customers in the upcoming days will have to clarify the problem and point them at a newer working version.

I don't have an opinion on adding ATS exceptions, though I tend to favor what ATS offers over developer convenience. Getting an SSL certificate is pretty easy nowadays.
