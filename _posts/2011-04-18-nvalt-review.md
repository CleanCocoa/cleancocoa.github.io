---
title: nvALT review
created_at: 2011-04-18 15:41 +0200
url: http://brettterpstra.com/project/nvalt/
image: "nvalt.png"
kind: worklog
tags: [nv, review]
---

If you are interested in taking notes fast and reliable but never got around NV's interface, now it's the best time to reevaluate the situation.  [Introducing nvALT][nvalt], the result of joint efforts of both [David Halter][et] and [Brett Terpstra][bt] who united their two popular NV forks.

## Praise ##

Two weeks ago, I switched from [my own NV fork][nvmmd] to [nvALT][nvalt].  In this newest community-customised fork of [Zachary Schneirov's original Notational Velocity][nv], a few neat design changes have been applied.  There is a fullscreen mode, widescreen-layout (`<3`), word count and a shortcut to open the current file in TextMate---just to name a few of my favorites.  Naturally, NV's original features introduced in January are included as well:  for example Tags are stored in OpenMeta format, meaning you can delete the `Notes & Settings` database file without any loss (except bookmarks), which initially make them useful for me since heavy NV customizations can render this file useless in no time.

Also, the **text width is adjustable**:  when set to 650px with a nice and free [Anonymous Pro 14pt][ano] font, you can resize the window to your liking but never exceed 78 characters.  In this way, I can focus on writing notes in fullscreen mode and still be able to manually break lines at a common width. (Common in respect of ancestral terminal use and derived guidelines for composing plain text files, mails and whatnot.)

[nvmmd]: http://christiantietze.de/zettelkasten/nv
[nv]: http://notational.net
[ano]: http://www.ms-studio.com/FontSales/anonymouspro.html

[nvalt]: http://brettterpstra.com/project/nvalt/
[et]: http://elasticthreads.tumblr.com
[bt]: http://brettterpstra.com/

## Issues ##

There are a few **issues** which I'd like to fix as soon as the source code is released.  

[![nvALT preview](http://farm6.static.flickr.com/5149/5630507279_a1fa9a72d5_o.png)](http://www.flickr.com/photos/divinedominion/5630507279)

1.  **The Markdown preview window style**.  It's default style is just 
    not-so-stylish at all but shows what can be accomplished when the user 
    takes some time to modify it.  I'd like to tweak the default template a 
    bit and enhance it attractivity-wise.
2.  **MultiMarkdown preview rendering ignores the whole header**.  Thus, 
    every note without a header is rendered quite nicely.  My fork had issues 
    with this: every document's first line or paragraph was assumed to be the 
    header, hence not rendered at all.  Now it's the opposite.  There is an 
    objectively right way just in the middle:  check for existing header 
    metadata.
3.  **Switch MultiMarkdown rendering engine.**  Fletcher himself released an 
    [incredibly fast rendering][pegmmd] engine earlier this year, on which I 
    [reported][rep].  I want that one, cutting edge, blazing fast 
    ([_live_ preview][live], that is).

Hereby, I **stall my fork** until nvALT's source becomes available.  Maybe I'm able to plug [my desired custiomizations][cust] into this new big thing.


[pegmmd]: https://github.com/fletcher/peg-multimarkdown/
[rep]: http://christiantietze.tumblr.com/post/3107859203/multimarkdown-3-0b1  TODO
[cust]: http://christiantietze.tumblr.com/post/1480460350/mass-edit-notes TODO
[live]: http://fletcherpenney.net/2011/04/multimarkdown_in_textmate_preview
