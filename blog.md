---
layout: default
title: Blog
permalink: /blog/
---

<section class="mb-[80px] max-w-3xl">
  <p class="mb-4 text-xs font-bold uppercase tracking-[0.3em] text-on-surface-variant">{{ site.title }}</p>
  <h1 class="mb-6 font-h1 text-h1 text-primary">Blog</h1>
  <p class="max-w-2xl text-body-lg text-on-surface-variant">
    {{ site.description }}
  </p>
</section>

{% comment %}
  Group posts by their top-level folder under _posts/.
  Posts at the root of _posts/ are treated as "uncategorized".
{% endcomment %}

{% assign root_posts = "" | split: "" %}
{% assign grouped_posts = "" | split: "" %}
{% assign folder_names = "" | split: "" %}

{% for post in site.posts %}
  {% assign parts = post.path | split: "/" %}
  {% comment %} parts: ["_posts", "Folder?", "...", "file.md"] {% endcomment %}
  {% if parts.size == 2 %}
    {% assign root_posts = root_posts | push: post %}
  {% else %}
    {% assign folder_name = parts[1] %}
    {% unless folder_names contains folder_name %}
      {% assign folder_names = folder_names | push: folder_name %}
    {% endunless %}
  {% endif %}
{% endfor %}

{% if root_posts.size > 0 %}
<section class="mb-16">
  <h2 class="mb-4 font-h3 text-h3 text-primary">Posts</h2>
  <div class="space-y-4">
    {% for post in root_posts %}
      <article class="group border border-surface-container-high bg-surface-container-lowest p-6 transition-colors duration-200 hover:border-neutral-900 dark:hover:border-neutral-50">
        {% include post_breadcrumb.html post=post %}
        <div class="mb-3 flex flex-wrap items-center gap-3 text-xs font-bold uppercase tracking-[0.24em] text-on-surface-variant">
          <span>{{ post.date | date: "%Y-%m-%d" }}</span>
          {% if post.city %}<span>{{ post.city }}</span>{% endif %}
          <span>{{ post.author | default: site.author }}</span>
        </div>
        <h3 class="mb-3 text-2xl font-semibold text-primary">
          <a href="{{ post.url | relative_url }}" class="inline-flex items-center gap-2">
            {{ post.title }}
            <span class="material-symbols-outlined text-base transition-transform duration-200 group-hover:translate-x-1">north_east</span>
          </a>
        </h3>
        <p class="text-body-md text-on-surface-variant">{{ post.excerpt | strip_html | truncate: 180 }}</p>
      </article>
    {% endfor %}
  </div>
</section>
{% endif %}

{% for folder_name in folder_names %}
<section class="mb-16">
  <h2 class="mb-4 font-h3 text-h3 text-primary">{{ folder_name }}</h2>
  <div class="space-y-4">
    {% for post in site.posts %}
      {% assign parts = post.path | split: "/" %}
      {% if parts.size > 2 and parts[1] == folder_name %}
      <article class="group border border-surface-container-high bg-surface-container-lowest p-6 transition-colors duration-200 hover:border-neutral-900 dark:hover:border-neutral-50">
        {% include post_breadcrumb.html post=post %}
        <div class="mb-3 flex flex-wrap items-center gap-3 text-xs font-bold uppercase tracking-[0.24em] text-on-surface-variant">
          <span>{{ post.date | date: "%Y-%m-%d" }}</span>
          {% if post.city %}<span>{{ post.city }}</span>{% endif %}
          <span>{{ post.author | default: site.author }}</span>
        </div>
        <h3 class="mb-3 text-2xl font-semibold text-primary">
          <a href="{{ post.url | relative_url }}" class="inline-flex items-center gap-2">
            {{ post.title }}
            <span class="material-symbols-outlined text-base transition-transform duration-200 group-hover:translate-x-1">north_east</span>
          </a>
        </h3>
        <p class="text-body-md text-on-surface-variant">{{ post.excerpt | strip_html | truncate: 180 }}</p>
      </article>
      {% endif %}
    {% endfor %}
  </div>
</section>
{% endfor %}