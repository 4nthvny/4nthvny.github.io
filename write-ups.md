---
title: Write-Ups
layout: page
---

# Writeups

Below are all my public cybersecurity writeups:

<ul>
  {% for post in site.posts %}
    {% if post.category == "writeups" %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endif %}
  {% endfor %}
</ul>
