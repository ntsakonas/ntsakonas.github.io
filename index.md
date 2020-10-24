---
layout: default
title: Home
---
<h2>Latest Post</h2>

{% for post in site.posts limit:1 %}
	<h2>{{ post.title }}</h2>
	<p>{{ post.date | date: "%b %-d, %Y" }}</p>

	{{ post.content }}
{% endfor %}


<h2>Recent Posts</h2>
{% for post in site.posts offset:1 limit:2 %}
	{{ post.excerpt }}
{% endfor %}