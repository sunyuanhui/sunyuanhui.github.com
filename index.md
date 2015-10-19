---
layout: page
title: Welcome to my blog!
tagline: A SWE's idealism!
---
{% include JB/setup %}

## Category List

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>



