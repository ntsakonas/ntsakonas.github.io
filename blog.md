---
layout: default
title: Blog
---
<h2>Latest Posts</h2>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}></a></h2>
      {{ page.date | date: "%b %-d, %Y" }}
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>