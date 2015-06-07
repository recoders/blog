--
layout: default
title: Life is code.
tagline: byte by byte
---
{% include JB/setup %}
    
<div class="posts">
  {% for post in site.posts %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ BASE_PATH }}/{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>

    <span class="post-date">{{ post.date | date_to_string }}</span>

    {{ post.content }}
  </div>
  {% endfor %}
</div>
