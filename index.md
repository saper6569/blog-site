---
layout: default
title: Home
---

<div class="home">
    <section class="intro">
        <h1>Welcome</h1>
        <p class="lead">
            I'm an Electrical Engineering student passionate about technology, innovation, and sharing knowledge 
            through technical writing and projects.
        </p>
    </section>

    <section class="quick-links">
        <h2>Quick Links</h2>
        <ul class="link-list">
            <li><a href="{{ '/projects' | relative_url }}">View My Projects</a> - Engineering projects and experiments</li>
            <li><a href="{{ '/blog' | relative_url }}">Read My Blog</a> - Technical articles and insights</li>
            <li><a href="https://github.com/{{ site.github_username }}" target="_blank" rel="noopener noreferrer">GitHub Profile</a> - Code repositories and contributions</li>
        </ul>
    </section>

    <section class="recent-posts">
        <h2>Recent Posts</h2>
        {% assign recent_posts = site.posts | limit: 5 %}
        {% if recent_posts.size > 0 %}
        <ul class="post-list">
            {% for post in recent_posts %}
            <li>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
            </li>
            {% endfor %}
        </ul>
        <p><a href="{{ '/blog' | relative_url }}">View all posts &rarr;</a></p>
        {% else %}
        <p>No posts yet. Check back soon!</p>
        {% endif %}
    </section>
</div>
