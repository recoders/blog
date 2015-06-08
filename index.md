---
layout: page
title: Life is code.
tagline: byte by byte
---
{% include JB/setup %}
    
## Последние записи

<div class="posts">
  {% for post in site.posts %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ BASE_PATH }}{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.date | date_to_string }}</span>

    {{ post.excerpt | remove: '<p>' | remove: '</p>' }}
  </div>
  {% endfor %}
</div>