---
layout: default
title: Blog
---
<h2>Latest Posts</h2>

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }} <i>{{ page.date | date: "%b %-d, %Y" }}</i></a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>