---
title: Nanoc3 Boilerplate
created_at: 2012-12-22 13:38:00 +0100
kind: worklog
url: https://github.com/DivineDominion/nanoc-boilerplate
tags: [deployment, git, nanoc]
---

I put my personal [nanoc](http://nanoc.stoneship.org) boilerplate setup on GitHub.  Maybe you find the deployment process useful.

## Deployment Process

I assume you're public html folder is called `htdocs/` and you can create new folders below your domains folder but outside `htdocs/`.

I also assume you use my Rakefile:  upon `rake build` it will checkout the branch 'deploy' and put all files from `output/` in there.  Uploading from 'deploy' to the production server will only copy the HTML output, not the nanoc setup.

1.  Initialize bare production git repository on the server:

        git init --bare ~/doms/example.com/git
2.  You'll want automatic updates when you push to the server.  Use git's own
    `post-receive` hook:

        # add to ~/doms/example.com/git/hooks/post-receive
        echo "Updating website ..."
        cd /the/full/path/to/doms/example.com/htdocs || exit
        unset GIT_DIR
        git pull origin 
        echo "Update complete."

    Make it executable:

        chmod +x post-receive

3.  Initialize git repository in `htdocs/`.  This will point to the bare
    repository on the server and check out the current version:
    
        # given you're in ~/doms/example.com/htdocs
        git init
        git remote add origin ../git
        
        # setup branch to pull from:
        git config branch.master.remote origin
        git config branch.master.merge refs/heads/deploy
4.  Setup production server locally:

        git remote add production ssh://user@example.com/~/doms/example.com/git/
        git remote show production
5.  Commit changes locally and put them on the server:
        
        git commit
        rake build
        git push production deploy
    
    You can push all branches via `git push production` to backup your code. 
    Only the branch 'deploy' will be visible to the public.

## Sources

I once combed this together from various sources:

<ul class="short">
    <li><a href="http://www.filosophy.org/post/33/dead_simple_git_deployment/">dead simple git deployment</a></li>
    <li><a href="http://www.deanoj.co.uk/programming/git/using-git-and-a-post-receive-hook-script-for-auto-deployment/">Using Git and a post-receive hook script for auto deployment</a></li>
    <li><a href="http://deadlytechnology.com/web-development/deploy-websites-with-git/">Deploy websites using Git - the easy way</a></li>
    <li><a href="http://danbarber.me/using-git-for-deployment/">Using Git for Deployment</a></li>
</ul>
