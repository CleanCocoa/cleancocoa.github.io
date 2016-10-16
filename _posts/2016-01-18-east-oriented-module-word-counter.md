---
title: I'll be writing a Word Counter Swift module fully East-Oriented
created_at: 2016-01-18 11:44:09 +0100
kind: worklog
tags: [ swift, module, east-oriented, async, bounded-context ]
image: 201601181145_analytics-sketch.jpg
comments: on
---

The Word Counter is my biggest software project so far, and it's growing. To add features, I discovered the use of Swift modules with joy. I can write code for all levels of the new feature in isolation in a new project that compiles fast. Later I plug the resulting project and module into the Word Counter, *et voilà*: feature finished.

{% include figure.md src="/assets/blog/2016/201601181145_analytics-sketch.jpg" alt="sketch of the analytics" caption="Initial sketch of the analytics module main interface" %}

For the next big update to the Word Counter I'm going to add an analytics module. It'll help look at progress and patterns. I have lots of very cool plans with it.

The analytics module will be different from the file monitoring module: it'll require an external data source. The file monitoring module I added last autumn came with its own persistence mechanism. It was fully self-contained except for the path to store the SQLite file. Analytics is made to display existing data -- data that the app measures for about two years now, and that has to be provided through a published interface.

I decided to design and code this module [East-Oriented][east] from start to finish. The file monitoring module exposes a few simple data points as properties, that means synchonously. The analytics module will interact with other components mostly through callbacks to factor asynchrony in early on. 

This is important because the module cannot form expectations about how the data it queries from the client is stores. Fetching data may take a while, require a round trip to a server, say, or long-running decompression of files on the hard disk. There's no way to tell. To design the inter-module interactions synchronously would be just stupid.

So East-Oriented it is, from the outermost layer deep down to the core of the module. I'm excited to see what this experiment will show. I guess it'll sometimes feel a bit extreme, but that's okay for this isolated module. Until now, I have always made compromises and mixed approaches when I thought it would benefit the overall design. The analytics module is simple enough to warrant the experiment. Will keep you posted.

[east]: /posts/tags/east-oriented/
