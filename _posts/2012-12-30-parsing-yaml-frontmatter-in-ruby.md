---
title: Parsing YAML Frontmatter in Ruby
created_at: 2012-12-30 12:07:00 +0100
kind: worklog
tags: [ruby, yaml]
---

I was trying to fix a non-issue with YAML by throwing Ruby gems at it.

[nanoc](http://nanoc.stoneship.org/) and [Jekyll](http://jekyllrb.com/) and the like provide YAML metadata for their content files:

    ---
    title:  Some page title
    ---
    
    # Heading #
    
    Lorem ipsum dolor sit amet, consectetur adipisicing 
    elit, sed do eiusmod tempor incididunt ut labore et 
    dolore magna aliqua.

Google said this is called **YAML Frontmatter**.  So I was looking at a way to parse YAML Frontmatter of plain text files, searching for gems -- until I found the [laconic answer on StackOverflow](http://stackoverflow.com/questions/3877004/how-do-i-parse-a-yaml-file):  _what's wrong with YAML's standard features?_

In the end, it's as easy as writing

    require 'yaml'
    thing = YAML.load_file('some.yml')
    puts thing.inspect

That returns just the YAML, the rest is ignored.
