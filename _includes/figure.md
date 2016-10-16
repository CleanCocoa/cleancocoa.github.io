<figure class="post-figure" markdown="1">
{% if include.url %}<a href="{{ include.url }}">{% endif %}
<img class="post-figure__image" src="{{ include.src }}" />
{% if include.url %}</a>{% endif %}

{% if include.caption %}
<figcaption markdown="1" class="post-figure__caption">
  {{ include.caption }}
</figcaption>
{% endif %}
</figure>
