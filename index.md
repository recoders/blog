---
layout: page
tagline: byte by byte
---
{% include JB/setup %}
    
<div class="posts">
  {% for post in site.posts %}
  {% assign author = site.data.authors[post.author] %}
  <div class="post">
    <h1 class="post-title">
      <a href="{{ BASE_PATH }}{{ post.url }}">
        {{ post.title }}
      </a>
      {% if author %} 
          <span class="post-author">{% if author.github %} <a href="//github.com/{{ author.github }}" target="_blank">{{ author.name }}</a>
              {% else %} <a href="{{ author.web }}" target="_blank">{{ author.name }}</a> {% endif %}
          </span>
      {% endif %}
    </h1>

    <span class="post-date">{{ post.date | date_to_string }}</span>

    {{ post.excerpt | remove: '<p>' | remove: '</p>' }}
  </div>
  {% endfor %}
</div>