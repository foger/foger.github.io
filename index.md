---
layout: default
title: Blog
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

{% for post in site.posts | sort post.date %}
<hr>
<article>
<h1>{{ post.title }}</h1>
{{ post.content }}
</article>
{% endfor %}
