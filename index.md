---
layout: home
---

{% for post in paginator.posts %}
  {% if post.pinned %}
    Pinned Post! 
    {{ post.title }}
    {{ post.date }}
    {{ post.content }}
  {% endif %}
{% endfor %}

