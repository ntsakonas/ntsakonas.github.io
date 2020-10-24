---
layout: default
title: Home
---
<h2>{{ site.posts.first.title }}</h2>
<p>{{ site.posts.first.date | date: "%b %-d, %Y" }}</p>

{{ site.posts.first.content }}