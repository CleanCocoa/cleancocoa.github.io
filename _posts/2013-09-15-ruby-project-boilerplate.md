---
title: Ruby RSpec/Guard/Spork Project Boilerplate
created_at: 2013-09-15 21:55:11 +0200
kind: worklog
tags: [ ruby ]

---

{% include figure.md src="/assets/blog/2013/201309152204_tmate.png" alt="Project template" url="https://github.com/DivineDominion/rspec-guard-spork-boilerplate" caption="Project template" %}

I got fed up with having to start a Ruby project from scratch every other day.  Thus I proudly present my [Ruby BDD project boilerplate][boil].

## Requirements

This is an "opinionated" boilerplate.  I say that because having an opinion seems to be _de rigueur_.  What I really want to say is:  I picked a setup which works for me and it might or might not work for you.

My development tools of choice include the [RSpec][] testing library with [guard][] as a modern replacement for the `autotest` gem.  Without automatted test execution doing TDD or BDD makes no sense.  To speed up test execution by magnitudes, I plug-in [Spork][] as a test suite pre-loader.  With guard and Spork, getting feedback fast is guaranteed.[^dis]

I use MRI Ruby v2.0 via [rvm][] and usually create a project-specific gemset when I know the project is going to be a fully-fledged gem or an application I eventually intend to ship.  I really dig recent Ruby version manager's feature to automatically switch between rubies and gemsets via the use of [`.ruby-version` and `.ruby-gemset`][autoload] files in the project root.  This feature made using project gemsets bearable in the first place.  I recommend you look into that stuff, too, but it's not necessary for this boilerplate code to be useful.  You can convert old `.rvmrc`-based projects with:

    rvm rvmrc to .ruby-version

For Rails applications, you might also want to have a look at:

- [Zeus][] preloads the Rails environment so that Rails-specific tasks take seconds only
- [Commands][] sidesteps loading the whole Rails environment

Ok.  Now go and check out my [Ruby RSpec-Guard-Spork boilerplate on GitHub][boil] right now.

  [boil]: https://github.com/DivineDominion/rspec-guard-spork-boilerplate
  [autoload]: http://rvm.io/workflow/projects#project-file-ruby-version
  [rvm]: http://rvm.io
  [rspec]: http://rspec.info/
  [guard]: http://guardgem.org/
  [spork]: https://github.com/sporkrb/spork
  [zeus]: https://github.com/burke/zeus
  [commands]: https://github.com/rails/commands

  [^dis]: Of course I do not literally guarantee fast test execution for your projects. But how catchy does the more approriate line "fast test execution is more likely" sound?  I'm no way responsible for bad OO design. Except when I am, because I suck at it sometimes. 
