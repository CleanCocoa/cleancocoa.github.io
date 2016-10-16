---
title: Remove Objective-C method declaration fluff with Keyboard Maestro
created_at: 2012-11-19 18:10:00 +0100
kind: worklog
tags: [objective-c, keyboard-maestro, macro]
---

I got the usual Objective-C method declaration:

    - (UITableViewCell *)tableView:(UITableView *)tableView
             cellForRowAtIndexPath:(NSIndexPath *)indexPath

When you reference this method, its proper name is `tableView:cellForRowAtIndexPath:`.  Removing all the parameter declarations and the return type by hand is really tedious.

I wrote a [Keyboard Maestro](http://www.keyboardmaestro.com) macro to remedy my pain:

{% include figure.md src="/assets/blog/2012/km-objc-strip.png" alt="Keyboard Maestro Macro" %}

The Regular Expression part, in more detail:

    (
        (?<=\:)      # look behind without for : without matching it,
        \([\w\* ]+\) # take method parameter types ...
        [\w]+[ ]?    # ... and names, including whitespace 
        |
        [\-\+ ]+     # match visibility operator and whitespace
        \([\w\* ]+\) # match method's return type
    )

Works like a charm.
