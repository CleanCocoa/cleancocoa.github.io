---
title: Edit HTML with TextEdit for Mac
created_at: 2014-07-12 13:29:07 +0200
kind: worklog
tags: [ mac ]
image: 201407121348_icon.png
comments: on
---

In June, I told you about how to customize nvALT's Markdown preview template so you can include images from a sub-directory of your notes' directory with ease. Turns out that you probably aren't able to edit the preview `.html` file in the first place without some effort. Here's how you configure TextEdit, Apple's own editor which comes with every Mac by default, to edit HTML files the way you need to.

If you followed the [instructions to change the preview template of nvALT][nv] until where you need to change the `basedir` to point to your note archive, you may open the `template.html` file with TextEdit and end up with this:

{% include figure.md src="/assets/blog/2014/201407121340_rendering.png" alt="TextEdit HTML rendering" caption="If you see this when you open HTML files in TextEdit, you have to change the settings. Don't save the file or you'll lose hidden parts of the code!" %}

TextEdit treats HTML files as it treats other rich text file types like Word documents:  it renders them in a [What You See Is What You Get][wysiwyg] fashion.  If you perform any changes now, you'll edit the visible part of the file.  To change the template, though, you will need to change (invisible) meta data in the code. Switching to plain text mode won't help.

There's a switch in TextEdit to stop _rendering_ HTML files and edit them visually. Apple's [support docs](http://support.apple.com/kb/ta20406) told me so. Open the preferences and switch to the "Open and Save" tab.  Turn "Display HTML files as HTML code instead of formatted text" _on_:

{% include figure.md src="/assets/blog/2014/201407121341_prefs.png" alt="TextEdit preferences" caption="Enable showing HTML code in TextEdit preferences" %}

Afterwards, you can edit the template and finally change the `basedir` in the header:

{% include figure.md src="/assets/blog/2014/201407121342_code.png" alt="TextEdit HTML code" caption="Now you'll see HTML in all its glory. Looking at the monospaced dullness, you may wonder: was it really worth it? Trust me. It was." %}

Thanks [@SemiraSK](http://twitter.com/SemiraSK) for pointing out the problem!

[nv]: /posts/2014/06/images-in-nvalt-notes/
[wysiwyg]: http://en.wikipedia.org/wiki/WYSIWYG
