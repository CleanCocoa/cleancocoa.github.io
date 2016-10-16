---
title: Archive
layout: default
permalink: /posts/
---

<ol class="posts">
{% for post in site.posts %}
<li class="posts__item">
  <span class="posts__item__date">{{ post.date | date_to_string }}</span> <a class="posts__item__link" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
</li>
{% endfor %}
</ol>
