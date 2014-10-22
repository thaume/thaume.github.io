---
layout: default
title: Talks
permalink: /talks/
---

<ul class="posts">
  {% for post in site.posts %}
    {% if post.talk %}
      <div class="post">
        <a href="{{ post.url | prepend: site.baseurl }}" class="post-link">
          <p class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</p>
          <h3 class="h2 post-title">{{ post.title }}</h3>
          <p class="post-summary">{{ post.meta.seo_description }}</p>
        </a>
      </div>
    {% endif %}   <!-- resource-p -->
  {% endfor %} <!-- page -->
</ul>
