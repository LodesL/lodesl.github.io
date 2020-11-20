---
layout: page
title: Blog
permalink: /blog/
---

Meine Blog-Posts. Hier schreibe ich ab und zu über verschiedene Themen des maschinellen Lernens.

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>