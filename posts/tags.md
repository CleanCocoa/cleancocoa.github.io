---
title: Tags
layout: page
permalink: /posts/tags/
---

{% capture tags %}
  {% for tag in site.tags %}
    {{ tag[0] }}
  {% endfor %}
{% endcapture %}
{% assign sortedtags = tags | split:' ' | sort %}

## Tags Overview

<ul class="alltags">
{% for tag in sortedtags %}
<li class="alltags__tag"><a href="#{{ tag | url_escape }}">{{ tag }}</a></li>
{% endfor %}
</ul>


## All Tagged Posts

{% for tag in sortedtags %}
  <h3 id="{{ tag | url_escape }}">{{ tag }}</h3>
  <ul class="tagged-posts">
  {% for post in site.tags[tag] %}
    <li class="tagged-post"><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
  <p><a href="#top">Back to top &uarr;</a></p>
{% endfor %}
