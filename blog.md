---
layout: default
title: Blog
---

# Latest Posts

  {% for post in site.posts %}

      ## [{{ post.title }}]({{ post.url }})
     
      {{ page.date | date: "%b %-d, %Y" }}
      {{ post.excerpt }}
  {% endfor %}
