---
title: Bringing Notational Velocity to a new level of massive text editing
created_at: 2010-11-04 18:17 +0100
kind: worklog
tags: [nv]
---

(Or should it read "Bringing massive text editing to a new level of Notational Velocity"?)

When working with **different notes** in a **similar context** at the **same time** in [Notational Velocity][nv], I might have a few ideas at the ready to tackle your needs.  And, even more importantly, _you_ might have thought about this problem as well and would like to offer suggestions or modifications to my approach so I might implement a new feature set to ease our pain.  In any way, this post might be just for you.

  [nv]: http://notational.net

## Ease the pain of working with multiple notes ##

Usually, I want to edit a bunch of notes during a distinct time:  There is a task I want to work on but which tackles more than one topic so I take notes in two separate lists as needed---or I use two notes right away, with one list and topic in each.  Or I'm in class and can not anticipate what the professor is going to elaborate on, thus different subtopics branch off of a main topic "root".

### Taking notes in class ###

When I take notes during the two hours a single session takes, two kinds of notes seem to repeat:

1.  Notes on a specific question;  the professor will explain the problem and
    ideally offers some possible conclusions.  The session evolves around this
    theme.
2.  Notes on "facts";  when explaining, the professor tackles a few related
    topics and influential persons from history to make the breadth of the topic
    approachable.  Also includes theory outlines, quotes and such.

Sometimes, I create a new note early and capture the session's topic/question only to find myself creating loosely related sub-notes for later reference, _bare information_ that might be of use later, in a totally different context---a _kick start_ when searching for, say, "philosophy of mind" in about a years time.  I can easily tackle connecting these notes via Hyperlinks.  But when I create a new note, the previously used search term or "filter" is gone.  That behaviour is intended.  It just does not prove to be usable in this situation.

### Connect notes with metadata ###

I can link from one note to another.  These are direct or **strong links**.  When not directly linking two notes in the content, one can **"loosely" connect** notes by the use of **tags**.  I do not and will never use Notational Velocity tags since I'd henceforth rely on NV's internal tagging capability.  But I want plain text: everlasting and portable without information loss.  So when talking about "tags", I mean a way to put keywords into a file itself. (Theoretically, both content or file name would work;  but for me, a file's content is the place to go.)

I could implement my own way to "tag":  write a artificial string at the end or beginning of the file (like `@foobar` or `#foobar` or the like) to virtually group notes when I search for that term right after creation.  I see a better way concerning my workflow:  [MultiMarkdown][mmd] offers metadata in its document header, so I will utilize existing features and put the tags right there.  The process goes as follows:

1.  Class begins.  Professor starts talking and introduces us to the topic.
2.  Create note A.  Insert "Keyword:  5-semester, week-4, philosophy-101" into
    the header section.  Save that line to clipboard for re-use.
3.  Create new note B for sub-topic.  Paste header keywords.
4.  Replace search term (= new note's title) with clipboard contents.
5.  A and B are found.  Repeat 3 to 5.

  [mmd]: http://fletcherpenney.net/multimarkdown


### Tell NV to remember I'm in a certain situation ###

My first approach is:  tell Notational Velocity that I'm in class for the next two hours and that probably every note I create in the meantime will be realated to this session.  This equals to NV inserting the Keyword line into new note's headers automatically.  Every note I create is loosely connected to other notes from class of this week by a term I can search for.  I call this **setting a context**.

This automates step 2 and 3.  But still I'd have to search for this session's term afterwards _manually_ to retrieve all notes in the same context, i.e to search for the _current context_, finding its _loosely connected notes_.


### Retrieve notes automatically ###

The fourth step can be automated as well.  This section is more like brainstorming:  I will present two ideas, only roughly sketched.  Both base not on:  (i) an automated re-search of the last term after creation, or (ii) a special "Create note without touching the search/title text field" dialogue, because the first option will come in your way most of the time you're _not_ in context-mode and the second option is a total discontinuation of Notational Velocity's simple interface principle.  Instead, I think of _two new ways to search in NV_.

### 1\. Two different search modes to switch between ###

After creation, the new note should still be focused on.  (That's what NV does currently.)  But instead of wasting the list to display only the current and all-new note, I still want to see the whole _context_.  There is one way which seems most pragmatic for me.  Via shortcut, switch between two search modes:

1.  normal search like it works just now,
2.  "context search" which selects all notes from my _current context_;
    the search field might be useless then, or maybe it is used to drill down
    one of the many notes I created during the last two hours.  Call this
    one **focus mode** and everyone should be thrilled right away.

### 2\. Using Nested Search & Creation ###

Another approach I shall call **nested searching**, that is performing a search on a previously found set of items.  Maybe you now the Twitter client [Tweetie for Mac][tw].  When you expand on a tweet/message or user profile, at the top of the window a navigation bar changes, just like [_breadcrumb navigation_][bread] on websites.

  [tw]: http://www.google.com/search?sourceid=chrome&ie=UTF-8&q=tweetie
  [bread]: http://www.alistapart.com/articles/whereami

NV uses this already, but limited to one level.  Search for a term which makes one or two notes appear.  Then select a note from the list with your keyboard or mouse.  See how the "search/title bar" changes its content?  The text it now contains is something different from a search term, hence the other results of your search do not disappear when you select something.  The text field is now in **title display mode**.  Click inside the list, but not on an item (i.e. below the last found item, in the gutter or white space).  The contents change back to your search term, so the text field is in **search term display mode**.

Imagine Tweetie's _breadcrumb navigation_ set to your current context.  This is (1) a search, but also (2) template-content for all notes you create while in that context.  If your context is not too specific, just add another level of _nested search_ (why limit to only one level if there can be infinite?) within your current context.  Applied to Tweetie, it is like looking at [@zen_habits][zh] timeline (list of all tweets) first, _focusing_ a specific conversation with someone (i.e. a reply to someone's tweet), which results in a navigation like this:

[![Tweetie Breadcrumb][img]][ln]

When you _create a new note while in a nested search_, the combined search terms will be put into the file as plain text to suffice current search criteria.  Requires _nested search_ and does not work with NV today, because one needs a way to tell NV to preserve previous search terms.  (Like hitting Shift-Enter will add a level of search instead of creating a new note with this name, clearing the text field afterwards.)

  [ln]: http://www.flickr.com/photos/divinedominion/5146401338/
  [img]: http://farm5.static.flickr.com/4104/5146401338_31e08ec450_o.png "Tweetie Breadcrumb"
  [zh]: http://twitter.com/zen_habits

## Concluding ##

Especially concerning _automated note retrieval_ for a current context may present a few problems.  All of this is the product of my own thinking and a short conversation with [Eddie Smith][prag][^up] and not at all thought out.  

  [prag]: http://www.practicallyefficient.com/2010/11/05/notational-velocity-tagging-without-the-tag-field/
  [^up]: Link updated, pointing to a new post of Eddie (2010-11-06).

Please send me your feedback via [mail](mailto:christian.tietze@gmail.com) or [Twitter @ctietze](http://twitter.com/ctietze)!
