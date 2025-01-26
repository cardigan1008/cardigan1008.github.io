---
layout: blog
title: "Blog"
permalink: /blog/
---

<h1>Welcome to My Blog</h1>
<p>Here you'll find my latest posts, updates, and thoughts on various topics. Feel free to explore!</p>

<ul>
  {% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <small>{{ post.date | date: "%B %d, %Y" }}</small>
  </li>
  {% endfor %}
</ul>
