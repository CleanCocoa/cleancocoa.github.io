---
title: UIStackView For Showing And Hiding User Interface Controls
created_at: 2016-07-03 13:34:18 +0200
kind: worklog
tags: [ autolayout ]
url: http://www.iosinsight.com/uistackview-for-showing-and-hiding-user-interface-controls/
comments: on
---

Animating and (de)activating Auto Layout constraints always leaves me puzzled at first. When on iOS, I try to not add constraints with explicit width/height values to train myself to rely on relative values. This helps me cover multiple screen sizes with the same settings.

I asked Lawrence MacFadyen on Twitter recently if his `UIStackView` animation wouldn't work without an explicit width. Seems it does, so he [published an updated post.](http://www.iosinsight.com/uistackview-for-showing-and-hiding-user-interface-controls/)
