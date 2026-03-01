---
layout: default
title: Blog
---

<div class="blog-index">
    <h1>Blog</h1>
    
    <p class="blog-intro">
        Technical articles, project write-ups, and engineering insights. Posts are listed in reverse 
        chronological order.
    </p>

    <ul class="post-list">
        {% for post in site.posts %}
        <li class="post-item">
            <article>
                <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
                <div class="post-meta">
                    <time datetime="{{ post.date | date: '%Y-%m-%d' }}">
                        {{ post.date | date: "%B %d, %Y" }}
                    </time>
                    {% if post.tags.size > 0 %}
                    <div class="post-tags">
                        {% for tag in post.tags %}
                        <span class="tag">{{ tag }}</span>
                        {% endfor %}
                    </div>
                    {% endif %}
                </div>
                {% if post.excerpt %}
                <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 50 }}</p>
                {% endif %}
            </article>
        </li>
        {% endfor %}
    </ul>

    {% if site.posts.size == 0 %}
    <p>No posts yet. Check back soon!</p>
    {% endif %}
</div>
