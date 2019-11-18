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

    {% if post.content contains "<!-- more -->" %}
      {{ post.content | split:"<!-- more -->" | first % }}
      <div style="text-align:right;">
        <a href="{{ post.url }}" class="btn"> Read More </a>
      </div>
    {% else %}
      {{ post.content }}
    {% endif %}

</article>
{% endfor %}
