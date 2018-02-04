---
layout: default
title: Infosec Musings
---
{{ title }}
<div id="home">
<h3>Blog Posts and Write-ups</h3>
<ul class="posts">
{% for post in site.posts %}
<li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</ul>
