---
layout: blog
title: "Blog"
permalink: /blog/
---

<h1>Latest Blog Posts</h1>
<p>Welcome to my blog! Here are my latest articles:</p>

<ul class="blog-posts">
  {% if site.posts.size > 0 %}
    {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      <small>Published on {{ post.date | date: "%B %d, %Y" }}</small>
    </li>
    {% endfor %}
  {% else %}
    <p>No posts found. Check your _posts folder.</p>
  {% endif %}
</ul>
