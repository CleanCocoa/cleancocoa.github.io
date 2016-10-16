---
title: gitlogger updated to deal with UTF-8
created_at: 2014-10-17 13:20:32 +0200
kind: worklog
tags: [ ruby, git, dayone ]
comments: on
---

For about two years now I [still](/posts/2013/06/gitlogger-improved/) use Brett Terpstra's [gitlogger][orig] to store my daily git activity in Day One.

As of late, it ceased to run properly. Turns out some messages to log contain fancy characters like real apostrophes, `’`. To fix this, add the following statement to the launchd configuration script:

    <key>EnvironmentVariables</key>
    <dict>
      <key>LANG</key>
      <string>en_US.UTF-8</string>
    </dict>

This will enable UTF-8 character encoding. In plain English, this means the script will be able to recognize such fancy characters where it previously choked and died.

For brevity, update your scripts in accordance to my [Gist on GitHub](https://gist.github.com/DivineDominion/5775355), or have a look at the code below:

<script src="https://gist.github.com/DivineDominion/5775355.js"></script>

Cheers to Brett for the initial version!

[orig]: http://brettterpstra.com/2012/05/08/scatterbrains-git-as-biographer/
