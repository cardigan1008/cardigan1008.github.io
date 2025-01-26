---
layout: blog
title: "Blog"
permalink: /blog/
---

<ul>
  {% if site.posts.size > 0 %}
    {% for post in site.posts %}
    <li>
        <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endfor %}
  {% else %}
    <p>No posts found. Check your _posts folder.</p>
  {% endif %}
</ul>
