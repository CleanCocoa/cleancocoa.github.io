---
title: How to Create Fixture Files for Unit Tests
created_at: 2016-03-10 07:52:38 +0100
kind: worklog
tags: [ testing, fixture, xcode ]
comments: on
preview: fulltext
---

This is mostly a reminder for my future self: It doesn't suffice to create a TXT file and add it to the target in the file inspector (⌘⌥1) to be able to read it as part of the bundle. You also have to drag it into the "Copy Bundle Resource" build phase of the target so it get, well, _bundled_ into the product. (In my case, it was the test target.)

{% include figure.md src="/assets/blog/2016/201603100753_bundle-txt.png" alt="screenshot of Xcode" caption="The .md file was part of the test target but loading it from the NSBundle failed until I added it to the proper Build Phase" %}

1. Create an empty file or add an existing file to the target's group in Xcode and make it part of the proper target,
2. drag it into "Copy Bundle Resources" build phase,
3. load it using `NSBundle(forClass: TheTestCase.self).URLForResource("filename", withExtension: "txt")`.

Storing PLIST or JSON files for structured data works just as well. I happen to require plain text flavors most of the time and always wonder why the loading fails for a couple of minutes.

