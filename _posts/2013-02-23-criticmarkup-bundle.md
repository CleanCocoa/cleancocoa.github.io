---
title: CriticMarkup and a TextMate Bundle
created_at: 2013-02-23 18:26:00 +0100
kind: worklog
tags: [git, github, textmate, plaintext, markup]
image: "criticmarkup.png"
---

After [CriticMarkup][cm] was released with a toolkit including Sublime Text 2 theme and commands, I simply ported the easy stuff to TextMate.  Since Sublime Text bundle files are heavily inspired by TextMate (to ensure compatibility with the popular Mac all-purpose editor, I suppose), this wasn't a very complicated task.

So there it is, my [CriticMarkup TextMate 2 bundle][cmtm].

  [cm]: http://criticmarkup.com/
  [cmtm]: https://github.com/DivineDominion/criticmarkup.tmbundle

Meanwhile, Hilton Lipschitz [wrote a TextMate 2 bundle himself][hil] which even supported actions and keyboard shortcuts to trigger editorial marks.  Since my inferior bundle repository was already incorporated into the [CriticMarkup toolkit][cmtk] and Hilton wasn't very attached to his repository, we merged our efforts and our repos.

  [hil]: http://www.hiltmon.com/blog/2013/02/15/criticmarkup-bundle-for-textmate-2/
  [cmtk]: https://github.com/CriticMarkup/CriticMarkup-toolkit

His additions weigh in a lot more than my foundation, and I'm really happy with the cool stuff he's coming up with.  This is a fun experince for I never worked with someone else on a code base via GitHub pull requests.

But in this post my main concern isn't the awesome fact that collaboration feels nice, but technical difficulties:

## Git Submodule issues

tl;dr
:   GitHub can't archive referenced submodules.  Gabe and Eric shouldn't rely on GitHub's download. [Jump to my conclusion.](<%=@item.path%>#2013-02-23_conclusion)

Git being as awesome as it is supports something called ["submodules"](http://book.git-scm.com/5_submodules.html).  With this technique, you can reference and check out other repos into a main repository.  The submodules can be updated independently, and when you're in a submodule's directory, git commands like `git pull` affect the submodule code only.

Basically, git submodules let you _import_ other repositories while keeping their repository structure intact.  The other, dirty way would be to download code and copy the files into your project, effectively cutting the cord to its origin.

I forked the CriticMarkup toolkit and included my `.tmbundle` repo as a submodule.

**Benefit:**  I can work on the 'official' TextMate bundle independently with collaborators and send pull requests to Gabe and Erik so they update their references.

**Downside:**  Downloading a Zip file from GitHub doesn't download any submodule's code.  So GitHub downloads are incompatible with git submodule functionality.

The only way to download the toolkit is from the command line, where one has to initailize the submodules after cloning the git repository:

    # either:
    git clone git://github.com/CriticMarkup/CriticMarkup-toolkit.git
    git submodule update --init
    
    # or:
    git clone --recursive git://github.com/CriticMarkup/CriticMarkup-toolkit.git

That's not very convenient.  So Gabe and Erik ponder whether they should simply include ordinary copies of the TextMate bundle files, just like all the other bundles in the Toolkit.  At least the zipped download would work this way and potential users are happy again.

There's another catch though.  De-referencing the submodule and including the files directly will make bundle contributions harder in the end:  there'll be not just one and only one way to merge each contributor's efforts.

1.  **Contribute to the TextMate Bundle:**  People contribute to the TextMate bundle by forking mine or Hilton's repo.  We can merge code afterwards and decide which is the main repository.  Also, the repository may be added to the [TextMate Bundle Index](http://textmate.org/), pushing changes to every user on updates.
    
2.  **Contribute to the Toolkit directly:**  People clone the Toolkit instead of the TextMate bundle and simpley modify the included `.tmbundle` subfolder.  They send pull requests to the toolkit repo, making changes public.

I'm happy with neither getting rid of the submodule or ruining the downloadable Zip file.  This will lead to a diversion in code bases which someone can only merge back manually into a canonical TextMate bundle (where by "canonical" I mean the bundle included into the Bundle Index).

## Resolving the issue the Open Source way

<span id="2013-02-23_conclusion"></span>
Giving up submodules is a pity, really, since using them for repository references just feels right and proves to be totally flexible.  But in the end it's not about the needs of developers but about making users happy.  There's need for other arguments.

I hope Gabe and Eric think the same, and that they think outside the box: GitHub isn't a platform suited to host downloads which are aimed at end-users.  It's a platform where developers meet and contribute to their code bases instead.

  [^1]: Some time ago, you could upload your own archives to GitHub.  That way, you could provide an easy-access zip file for anyone while leaving the real thing for the hackers.  It's a manual process, still.  And most of the time, you would provide archives time and again, not on every single commit.

The download link at [criticmarkup.com][cm] _could_ but probably _shouldn't_ point to GitHub.  It's not GitHub's job to provide usable (binary) builds.  It's only coincidence that CriticMarkup doesn't need any compilation whatsoever, it's not the norm.

I propose another workflow instead:  Gabe and Eric should **provide automatic repository archiving on their own**.

* Clone the GitHub repo to a local machine, another server, or to [criticmarkup.com][cm] itself, then
* Set up maybe nightly updates via `cron` to perform a `git pull`.
* On every update, initialize and update any submodules and
* Archive the files in a snapshot not unlike nightly builds.
* Link to this nightly build-edition instead.

Of course I frown at the complexity of this approach for something so small.  But then again, this is exactly how Open Source software has been developed and distributed for ages.
