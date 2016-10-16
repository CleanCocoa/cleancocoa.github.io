---
title: Generate interactive documentation with Code Guide
created_at: 2013-06-06 13:26:55 +0200
kind: worklog
tags: [ programming, markdown, documentation, ruby, python, java ]
---

Nat Pryce released [Code Guide][cg], a tool to create interactive code documentation.  See [his blog post announcement.][np]  It's written in Python and works for Python and Java code.  But since Python and Ruby comments look the same, parsing Ruby code works, too.

[Check out one of the examples][ex] to see what the output looks like.  There are other examples [on Nat's blog.][np]

I really like these already but wonder how long tutorials would work, where paragraphs of text, section headings and code snippets flow in a single document.

*Code Guide* uses Markdown to markup the documentation.  You also have to mark comments as *Code Guide*-style with a pipe:

    # Python example. (This line will not be processed by Code Guide)
    #| This is the start
    some_code()
    #|.

I'll have to play with this for some time longer to see whether code becomes unbearable cluttered with exhaustive documentation texts.

That the documentation is included in the code also is the nice thing about this project:  Nat found it is infeasible to write documentation and tutorials containing code examples in a place outside of the code itself.  You'd have to maintain both the code base and the example code listings, keeping them in sync.  _Code Guide_ does solve this issue.

  [np]: http://www.natpryce.com/articles/000798.html
  [cg]: https://github.com/npryce/code-guide
  [ex]: http://www.natpryce.com/software/code-guide/example/selector-button-blink.html
  
