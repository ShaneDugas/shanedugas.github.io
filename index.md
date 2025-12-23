---
layout: default
title: "Home"
---

<ul class="post-list">
  {% for post in site.posts %}
  <li class="post-list-item">
    <div class="post-date">{{ post.date | date: "%Y-%m-%d" }}</div>
    <a class="post-link" href="{{ post.url | relative_url }}">
      <div class="post-title">{{ post.title }}</div>
    </a>
    {% if post.excerpt %}
    <div class="post-excerpt">
      {{ post.excerpt | strip_html | truncate: 220 }}
    </div>
    {% endif %}
  </li>
  {% endfor %}
</ul>

