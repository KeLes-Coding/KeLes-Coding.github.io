---
layout: default
title: Blog
permalink: /blog/
---

<section class="mb-[40px] max-w-3xl">
  <p class="mb-4 text-xs font-bold uppercase tracking-[0.3em] text-on-surface-variant">{{ site.title }}</p>
  <h1 class="mb-6 font-h1 text-h1 text-primary">Blog</h1>
  <p class="max-w-2xl text-body-lg text-on-surface-variant">
    {{ site.description }}
  </p>
</section>

{% comment %}
  Collect folder names and categorize posts.
{% endcomment %}

{% assign root_posts = "" | split: "" %}
{% assign folder_names = "" | split: "" %}

{% for post in site.posts %}
  {% assign parts = post.path | split: "/" %}
  {% if parts.size == 2 %}
    {% assign root_posts = root_posts | push: post %}
  {% else %}
    {% assign folder_name = parts[1] %}
    {% unless folder_names contains folder_name %}
      {% assign folder_names = folder_names | push: folder_name %}
    {% endunless %}
  {% endif %}
{% endfor %}

<div class="flex flex-col gap-8 md:flex-row md:gap-12">
  <aside class="md:w-56 md:shrink-0">
    <nav class="space-y-4">
      <div class="space-y-1">
        {% if root_posts.size > 0 %}
        <button class="blog-folder-btn w-full cursor-pointer text-left text-sm font-bold uppercase tracking-[0.16em] text-on-surface-variant transition-colors duration-200 hover:text-primary py-1" data-folder="posts">
          Posts
        </button>
        {% endif %}
        {% for folder_name in folder_names %}
        <button class="blog-folder-btn w-full cursor-pointer text-left text-sm font-bold uppercase tracking-[0.16em] text-on-surface-variant transition-colors duration-200 hover:text-primary py-1 {% if forloop.first and root_posts.size == 0 %}text-primary{% endif %}" data-folder="{{ folder_name | slugify }}">
          {{ folder_name }}
        </button>
        {% endfor %}
      </div>
    </nav>
  </aside>

  <div class="min-w-0 flex-1">
    {% if root_posts.size > 0 %}
    <div class="blog-post-group" id="folder-posts">
      <div class="space-y-4">
        {% for post in root_posts %}
        <article class="group border border-surface-container-high bg-surface-container-lowest p-6 transition-colors duration-200 hover:border-neutral-900 dark:hover:border-neutral-50">
          {% include post_breadcrumb.html post=post %}
          <div class="mb-3 flex flex-wrap items-center gap-3 text-xs font-bold uppercase tracking-[0.24em] text-on-surface-variant">
            <span>{{ post.date | date: "%Y-%m-%d" }}</span>
            {% if post.city %}<span>{{ post.city }}</span>{% endif %}
            <span>{{ post.author | default: site.author }}</span>
          </div>
          <h3 class="text-2xl font-semibold text-primary">
            <a href="{{ post.url | relative_url }}" class="inline-flex items-center gap-2">
              {{ post.title }}
              <span class="material-symbols-outlined text-base transition-transform duration-200 group-hover:translate-x-1">north_east</span>
            </a>
          </h3>
        </article>
        {% endfor %}
      </div>
    </div>
    {% endif %}

    {% for folder_name in folder_names %}
    <div class="blog-post-group {% unless forloop.first and root_posts.size == 0 %}hidden{% endunless %}" id="folder-{{ folder_name | slugify }}">
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
          <h3 class="text-2xl font-semibold text-primary">
            <a href="{{ post.url | relative_url }}" class="inline-flex items-center gap-2">
              {{ post.title }}
              <span class="material-symbols-outlined text-base transition-transform duration-200 group-hover:translate-x-1">north_east</span>
            </a>
          </h3>
        </article>
          {% endif %}
        {% endfor %}
      </div>
    </div>
    {% endfor %}

    <div class="blog-post-group hidden" id="folder-search-results">
      <p class="text-on-surface-variant text-body-md" id="search-no-results">No matching posts found.</p>
      <div class="space-y-4" id="search-results-list"></div>
    </div>
  </div>
</div>

<script>
(function() {
  const btns = document.querySelectorAll('.blog-folder-btn');
  const groups = document.querySelectorAll('.blog-post-group');
  const searchInput = document.getElementById('blog-search');

  function showFolder(folderId) {
    groups.forEach(function(g) { g.classList.add('hidden'); });
    var el = document.getElementById('folder-' + folderId);
    if (el) el.classList.remove('hidden');
    btns.forEach(function(b) {
      if (b.dataset.folder === folderId) {
        b.classList.add('text-primary');
      } else {
        b.classList.remove('text-primary');
      }
    });
    if (searchInput) searchInput.value = '';
  }

  btns.forEach(function(btn) {
    btn.addEventListener('click', function() {
      showFolder(btn.dataset.folder);
    });
  });

  {% if root_posts.size > 0 %}
  showFolder('posts');
  {% else %}
  {% for folder_name in folder_names %}
  {% if forloop.first %}
  showFolder('{{ folder_name | slugify }}');
  {% endif %}
  {% endfor %}
  {% endif %}

  var searchData = [
    {% for post in site.posts %}
    {
      title: {{ post.title | jsonify }},
      url: {{ post.url | relative_url | jsonify }},
      date: {{ post.date | date: "%Y-%m-%d" | jsonify }},
      {% if post.city %}city: {{ post.city | jsonify }},{% endif %}
      author: {{ post.author | default: site.author | jsonify }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  if (searchInput) {
    searchInput.addEventListener('input', function() {
      var q = this.value.trim().toLowerCase();
      groups.forEach(function(g) { g.classList.add('hidden'); });
      btns.forEach(function(b) { b.classList.remove('text-primary'); });

      if (!q) {
        {% if root_posts.size > 0 %}
        showFolder('posts');
        {% else %}
        {% for folder_name in folder_names %}
        {% if forloop.first %}
        showFolder('{{ folder_name | slugify }}');
        {% endif %}
        {% endfor %}
        {% endif %}
        return;
      }

      var results = searchData.filter(function(p) {
        return p.title.toLowerCase().indexOf(q) !== -1;
      });

      var noResults = document.getElementById('search-no-results');
      var resultsList = document.getElementById('search-results-list');
      var resultsPanel = document.getElementById('folder-search-results');

      resultsList.innerHTML = '';
      if (results.length === 0) {
        noResults.classList.remove('hidden');
      } else {
        noResults.classList.add('hidden');
        results.forEach(function(p) {
          resultsList.innerHTML += '<article class="group border border-surface-container-high bg-surface-container-lowest p-6 transition-colors duration-200 hover:border-neutral-900 dark:hover:border-neutral-50">' +
            '<div class="mb-3 flex flex-wrap items-center gap-3 text-xs font-bold uppercase tracking-[0.24em] text-on-surface-variant">' +
              '<span>' + p.date + '</span>' +
              (p.city ? '<span>' + p.city + '</span>' : '') +
              '<span>' + p.author + '</span>' +
            '</div>' +
            '<h3 class="text-2xl font-semibold text-primary">' +
              '<a href="' + p.url + '" class="inline-flex items-center gap-2">' +
                p.title +
                '<span class="material-symbols-outlined text-base transition-transform duration-200 group-hover:translate-x-1">north_east</span>' +
              '</a>' +
            '</h3>' +
          '</article>';
        });
      }
      resultsPanel.classList.remove('hidden');
    });
  }
})();
</script>