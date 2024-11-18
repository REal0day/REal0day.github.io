---
layout: default
permalink: /blog/
title: blog
nav: true
nav_order: 1
pagination:
  enabled: true
  collection: posts
  per_page: 5
  permalink: /page/:num/
  sort_field: date
  sort_reverse: true
  trail:
    before: 1
    after: 3
---

<div class="blog-index">
  <div class="header-bar">
    <h1>{{ site.blog_name }}</h1>
    <h2>{{ site.blog_description }}</h2>
  </div>
<div class="post">

{% assign blog_name_size = site.blog_name | size %}
{% assign blog_description_size = site.blog_description | size %}

{% if blog_name_size > 0 or blog_description_size > 0 %}

  <div class="header-bar">
    <h1>{{ site.blog_name }}</h1>
    <h2>{{ site.blog_description }}</h2>
  </div>
  {% endif %}


<div class="tag-category-list">
  <h2>Tags</h2>
  <ul>
    {% for tag in site.tags %}
      <li>
        <a href="/blog/tag/{{ tag[0] | slugify }}/">{{ tag[0] }}</a>
      </li>
    {% endfor %}
  </ul>


  <ul class="post-list">
    {% for post in paginator.posts %}
      <li class="post-item">
        <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
        <p>{{ post.description }}</p>
        <p class="post-meta">
          Posted on {{ post.date | date: '%B %d, %Y' }}
          {% if post.tags %}
            &middot; Tags:
            {% for tag in post.tags %}
              <a href="/blog/tag/{{ tag | slugify }}/">{{ tag }}</a>{% unless forloop.last %}, {% endunless %}
            {% endfor %}
          {% endif %}
        </p>
      </li>
    {% endfor %}
  </ul>
  
<div class="pagination">
  {% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path | relative_url }}">&laquo; Previous</a>
  {% endif %}

  {% for post in paginator.posts %}
    <!-- Render post -->
    <li>
      <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
      <p>{{ post.description }}</p>
      <p>Posted on {{ post.date | date: '%B %d, %Y' }}</p>
    </li>
  {% endfor %}

  {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path | relative_url }}">Next &raquo;</a>
  {% endif %}
</div>
