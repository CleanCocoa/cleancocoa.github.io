---
title: On Proprietarity of File Formats
created_at: 2016-09-19 08:05:18 +0200
kind: worklog
tags: [ proprietarity, persistence ]
vgwort: http://vg01.met.vgwort.de/na/38ae79437a7243d3806c6d1233ddd28b
comments: on
---

The latin root of "proprietary" indicates this is something someone created in a special way. We can say it is owned by someone, intellectually. But then again, every file format was conceived by a human being; so the origin alone doesn't suffice to grasp this concept.

Take a look at the general introduction of ["Proprietary format" on Wikipedia][wiki]. Accessed today (2016-09-18), it reads (highlighting mine):

> A proprietary format is a file format \[...\] that contains data that is ordered and stored according to a particular encoding-scheme, **designed \[...\] to be secret,** such that the decoding and interpretation of this stored data is only easily accomplished with particular software or hardware that the company itself has developed. **The specification of the data encoding format is not released**, or underlies non-disclosure agreements. A proprietary format can also be a file format whose encoding is in fact published, but is restricted through licences such that only the company itself or licencees may use it. **In contrast, an open format is a file format that is published and free to be used by everybody.**

In a nutshell, it is ...

* a secret file format: only people from the original source know how to use it;
* based on (usually unpublished) specifications;
* not meant to be used by everybody.

These are different things. Let's explore them further.

I can use XML or JSON and come up with a specification for my apps data storage. If I don't publish the specs, how would you know how to use it? It's plain text, so you can reverse-engineer the specification. But that won't be 100% reliable; also, app updates could break your interpretation of my format since I don't promise to not make changes.

A fantasy specification like this is not open. When the specs aren't open, no 3rd party can rely on the format. When nobody can rely on the format except the inventor, then this format is closed for use.

Open formats, as cited above, are based on published specs _and_ free to use.

Even without any documentation, a short notice of not intending to break the current version of the format can help 3rd party developers build programs using a format. Their reverse-engineering then is more likely to stay functional. "Freezing" the specification this way facilitates growth of an ecosystem around that format, even if the spelled-out specs are only available inside the head of the inventor. Without freezing the format, we cannot safely assume that we're welcome as a 3rd party. Is a frozen but private format proprietary or not?

Here's a suggestion to differentiate "open format" further:

* **Technological openness:** can you open the file with a standard editor and read it? Applies to plain text-based formats, but also SQLite database files. Reverse-engineering binary formats with HEX editors or similar isn't very fun.
* **Intentional openness:** can you safely assume that the author wants you to work with the data, read it and write your own? Or is it likely the author will consider you an intruder?

I suggest the term "proprietary" to be used for formats even if they are based on plain text, when the _intention_ is to keep the specs to oneself.

* Markdown is an open format: it is technologically and intentionally open. This has helped a lot to create an ecosystem around the format. (Same with reStructuredText and Textile, both less popular, though.)
* Day One v1 used XML, technologically open, but intentionally ... not so much. It was stable, but only accidentally, not intentionally. There were 3rd party tools to create Day One journal entries using scripts, which was fun and probably added to the appeal of the app. (Day One v2 switched to a fully closed format.)
* Apple's office software (formally called iWork) uses technologically and intentionally closed formats: `.pages`, `.keynote`, `.numbers`. (I'd love to make Pages' files monitorable in the Word Counter, but it seems I'm out of luck here.)

It took the world a while to get a published specification for PDFs, Photoshop files, and the like. Chances are the applications you use each have their own file format to make storing data easy for developers and to provide some extra features.

So even if it's easy to use someone else's (i.e. "owned") custom file format, it doesn't automatically mean that it's open. Like library and API design for public use, designing file formats for use by the general public requires a different mindset of responsibility. Honestly, I wouldn't want that responsibility, either, if that means limiting my options to improve my software. (I guess that's why creating Day One v1 journal entries did work well but wasn't officially documented.)

* * * *

Narrowing down to text-based user data: in the end, only plain text is really durable. Day One v1 stored journal entries in XML files that added very easy to understand metadata to Markdown body text. I'd say YAML front matter ([see an example](https://middlemanapp.com/basics/frontmatter/)) would've worked even better in terms of data portability, but basic XML was okay. People could easily create an exporter on their own.

Plain text is also a format where users can screw up the data easily. Nobody would open a PDF in a text editor to make changes and destroy file integrity. Sticking to the example of YAML front matter, it's easy to introduce parsing errors accidentally when you have lines like:

    title: New idea: break this parser

Which would have to read:

    title: "New idea: break this parser"

... so the parser knows where the meta data key (`title`) ends and the value begins.

I design my software around plain text and use TXT files as an API. (Most of these projects you don't know, yet.) Because then 3rd party developers can chime in and provide shortcuts, overviews, or otherwise participate in the ecosystem of handling textual data. It's more work than coming up with my own format.

There's a good reason that Excel files are so complex; or that Numbers doesn't store files in CSV format. The advanced features don't map to simple textual representations easily. On the downside, Numbers changed file formats in recent years, and so did MS Excel. Microsoft's format is _owned_, but it's [_published_](https://msdn.microsoft.com/en-us/library/dd922181(v=office.12).aspx) and they clearly want developers to build upon it. 

The thing is: for most people, a textual representation would be sufficient. Calculate a sum of values in a column doesn't require a sophisticated file format. (Few business use cases stop here, I admit, but lots of private use cases do.) This could be done with a few clever conventions, maybe using MultiMarkdown table syntax to have prettier layout.

* * * *

Plain text "databases" can be used on any platform (ignoring file encodings for the sake of argument) and with any editor. Migrating from one app to another is easy. (And oftentimes not encouraged, of course; vendor lock-in can be a tool.) If an app states that it cares about you, your data, your life, family, dogs and cats, it better use a widely adopted file format. Open formats are even better, plain text being the king of all.

[wiki]: https://en.wikipedia.org/wiki/Proprietary_format
