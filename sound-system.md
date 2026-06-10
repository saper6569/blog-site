---
layout: default
title: Sound System
---

# Sound System


# Posts

<div class="post-grid">
  {% for post in site.tags["Sound System"] %}
    <a class="post-card" href="{{ post.url | relative_url }}">
      {% if post.image %}
        <img src="{{ post.image | relative_url }}" alt="{{ post.title }}">
      {% endif %}


  <div class="post-card-content">
    <h2>{{ post.title }}</h2>
    
    {% if post.excerpt %}
      <p>{{ post.excerpt | strip_html | truncate: 120 }}</p>
    {% endif %}
    
    <span class="read-more">Read More →</span>
  </div>
</a>


{% endfor %}

</div>
