---
layout: default
title: Blog
---
<h2>All Posts</h2>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.date | date: "%b %-d, %Y" }}
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>