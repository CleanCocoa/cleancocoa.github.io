---
title: gitlogger Improved
created_at: 2013-06-13 19:34:22 +0200
kind: worklog
tags: [ ruby, git, dayone ]
vgwort: http://vg02.met.vgwort.de/na/de61ab57bec540d0ab2716c4f4bc9b93
---

I use Brett Terpstra's little Ruby script called "[gitlogger][orig]" to write git commit messages from selected repositories into my Day One journal once a day.

My commit messages tend to be a bit longer when I work on projects which really matter.  Unfortunately, _gitlogger_ wasn't intended to handle multi-line commit messages.  Every commit message resides in a list item.  But if you know Markdown, you'll know this markup won't render as expected:

    Git Log 2013-06-13:

    * **[imprTheme]** 15:41: adds pingback template (59c2959)
    - pingbacks at the bottom have another list
    - ... take up less space
    - ... are in blockquotes

    * **[imprTheme]** 15:32: deletes .min.css on cleanup (6137ad4)


The list of commit messages won't contain two items but six, because the sub-list isn't recognized as such.  It has to be indented by four spaces to create a nested list.  Expected output would be:

    Git Log 2013-06-13:
    
    *   **[imprTheme]** 15:41: adds pingback template (59c2959)
        - pingbacks at the bottom have another list
        - ... take up less space
        - ... are in blockquotes
    
    *   **[imprTheme]** 15:32: deletes .min.css on cleanup (6137ad4)

I just got upset enough with my git commit messages being rendered poorly in [Day One][do] to finally change [Brett's script (at rev. 5)][bgist].  Now every message is indented in a Markdown-friendly way.

{% include figure.md src="/assets/blog/2013/201306131943_gitlogger_dayone.png.png" alt="Gitlogger output in Day One as expected" %}

For installation instructions, see [Brett's original post][orig].

[bgist]: https://gist.github.com/ttscoff/2632346/ccfcae43f8330c53da139efbd715e75f88764575
[orig]: http://brettterpstra.com/2012/05/08/scatterbrains-git-as-biographer/
[gist]: https://gist.github.com/DivineDominion/5775355
[do]: http://dayoneapp.com/
