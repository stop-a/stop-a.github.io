---
layout: default
title: Infosec Musings
---
{{ title }}
<div id="home">
# Blog Posts
<ul class="posts">
{% for post in site.posts %}
<li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
