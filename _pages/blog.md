---
layout: default
permalink: /blog/
title: Blog
nav: true
nav_order: 1
---

<div class="post">

  {% assign blog_name_size = site.blog_name | size %}
  {% assign blog_description_size = site.blog_description | size %}

  {% if blog_name_size > 0 or blog_description_size > 0 %}
    <div class="header-bar">
      <h1>{{ site.blog_name }}</h1>
      <h2>{{ site.blog_description }}</h2>
    </div>
  {% endif %}

  {% if site.display_tags.size > 0 or site.display_categories.size > 0 %}
    <div class="tag-category-list">
      <ul class="p-0 m-0">
        {% for tag in site.display_tags %}
          <li>
            <i class="fa-solid fa-hashtag fa-sm"></i> <a href="{{ tag | slugify | prepend: '/blog/tag/' | relative_url }}">{{ tag }}</a>
          </li>
          {% unless forloop.last %}
            <p>&bull;</p>
          {% endunless %}
        {% endfor %}
        {% if site.display_categories.size > 0 and site.display_tags.size > 0 %}
          <p>&bull;</p>
        {% endif %}
        {% for category in site.display_categories %}
          <li>
            <i class="fa-solid fa-tag fa-sm"></i> <a href="{{ category | slugify | prepend: '/blog/category/' | relative_url }}">{{ category }}</a>
          </li>
          {% unless forloop.last %}
            <p>&bull;</p>
          {% endunless %}
        {% endfor %}
      </ul>
    </div>
  {% endif %}

  <ul class="post-list">
    {% assign postlist = paginator.posts %}
    {% for post in site.posts %}
      <li>
        <h3>
          <a class="post-title" href="{{ post.url | relative_url }}">{{ post.title }}</a>
        </h3>
        <p>{{ post.description }}</p>
        <p class="post-meta">
          {{ post.date | date: '%B %d, %Y' }}
        </p>
      </li>
    {% endfor %}
  </ul>

  {% if paginator.total_pages > 1 %}
    <nav class="pagination">
      {% if paginator.previous_page %}
        <a href="{{ paginator.previous_page_path | relative_url }}" class="prev">Previous</a>
      {% endif %}

      {% for page in (1..paginator.total_pages) %}
        <a href="{{ paginator.page_path | relative_url | replace: ':num', page }}" class="{% if page == paginator.page %}active{% endif %}">{{ page }}</a>
      {% endfor %}

      {% if paginator.next_page %}
        <a href="{{ paginator.next_page_path | relative_url }}" class="next">Next</a>
      {% endif %}
    </nav>
  {% endif %}

</div>
