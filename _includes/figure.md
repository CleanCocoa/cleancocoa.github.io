<figure markdown="1">
{% if include.url %}<a href="{{ include.url }}">{% endif %}
<img src="{{ include.src }}" />
{% if include.url %}</a>{% endif %}

{% if include.caption %}
<figcaption markdown="1">
  {{ include.caption }}
</figcaption>
{% endif %}
</figure>
