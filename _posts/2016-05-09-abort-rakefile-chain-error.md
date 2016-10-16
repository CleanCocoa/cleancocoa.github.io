---
title: How to Abort a Chain of Rake Tasks on Error
created_at: 2016-05-09 14:25:00 +0200
kind: worklog
tags: [ ruby, website ]
comments: on
---

I use [nanoc](http://nanoc.ws/) as my static site creator for this blog. It's written in Ruby, my favorite scripting language. And so I use a `Rakefile` to automate most things, like generating a fresh copy of the site and deploying it to my server.

Only last week did I find out how to make Rake _not_ continue when a part of its tasks failed. Most of the stuff I use is wrappers around shell commands with a few system notifications sprinkled in. `$?` does capture the latest shell call's return value (kudos [dnsimple.com](https://blog.dnsimple.com/2016/04/publish-static-via-travis-to-cloudfront/)): 

    #!ruby
    task :generate do
      out = system "nanoc co" # compile site
      
      # abort on error
      if $?.to_i == 0
        puts  "Compilation succeeded"
      else
        abort "Compilation failed: #{$?.to_i}\n#{out}\n"
      end
    end

In the past, when I changed something and was to lazy to preview changes, the task chain of cleaning + compilation + deployment sometimes resulted in compilation errors; then a lot of files would be missing in the compilation result's folder; and then the folder would still get `rsync`ed to my server, deleting most of the stuff there. Oh my. I'm glad these times are gone. Astonishingly, total data loss on the server never bugged me enough to look this simple thing up.
