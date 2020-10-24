---
layout: default
title: Home
---
<h2>Latest Post</h2>

<h2>{{ site.posts.last.title }}</h2>
<p>{{ site.posts.last.title.date | date: "%b %-d, %Y" }}</p>

{{ site.posts.last.title.content }}