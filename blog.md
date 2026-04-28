---
layout: default
title: Blog
permalink: /blog/
---

<section class="mb-[40px] max-w-3xl">
  <p class="mb-4 text-xs font-bold uppercase tracking-[0.3em] text-on-surface-variant">{{ site.title }}</p>
  <h1 class="mb-6 font-h1 text-h1 text-primary">Blog</h1>
</section>

{% assign visible_posts = site.posts | where_exp: "post", "post.visible != false" %}

<div class="flex flex-col gap-8 md:flex-row md:gap-12">
  <aside class="md:w-64 md:shrink-0">
    <nav class="blog-tree" id="blog-folder-tree" aria-label="Blog folders"></nav>
  </aside>

  <div class="min-w-0 flex-1">
    <div class="mb-5 flex flex-wrap items-end justify-between gap-3 border-b border-surface-container-high pb-4">
      <div>
        <p class="text-xs font-bold uppercase tracking-[0.18em] text-on-surface-variant" id="blog-folder-kicker">All posts</p>
        <h2 class="mt-1 text-2xl font-semibold text-primary" id="blog-folder-title">All posts</h2>
      </div>
      <span class="text-sm text-on-surface-variant" id="blog-folder-count"></span>
    </div>

    <div class="space-y-4" id="blog-post-list"></div>

    <div class="hidden" id="blog-empty-state">
      <p class="text-on-surface-variant text-body-md">No matching posts found.</p>
    </div>
  </div>
</div>

<script>
(function() {
  var posts = [
    {% for post in visible_posts %}
    {% assign parts = post.path | split: "/" %}
    {% assign folder_segments = "" | split: "" %}
    {% for segment in parts %}
      {% if forloop.index0 > 0 and forloop.last == false %}
        {% assign folder_segments = folder_segments | push: segment %}
      {% endif %}
    {% endfor %}
    {
      title: {{ post.title | jsonify }},
      url: {{ post.url | relative_url | jsonify }},
      date: {{ post.date | date: "%Y-%m-%d" | jsonify }},
      city: {{ post.city | default: "" | jsonify }},
      author: {{ post.author | default: site.author | jsonify }},
      folders: [{% for folder in folder_segments %}{{ folder | jsonify }}{% unless forloop.last %},{% endunless %}{% endfor %}]
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  var treeEl = document.getElementById('blog-folder-tree');
  var listEl = document.getElementById('blog-post-list');
  var emptyEl = document.getElementById('blog-empty-state');
  var titleEl = document.getElementById('blog-folder-title');
  var kickerEl = document.getElementById('blog-folder-kicker');
  var countEl = document.getElementById('blog-folder-count');
  var searchInput = document.getElementById('global-search');

  function makeNode(name) {
    return { name: name, children: [], childMap: Object.create(null), posts: [], count: 0, oldestDate: '' };
  }

  function extractOrderNumber(text) {
    var match = String(text || '').match(/(?:^|[^0-9])(\d+)\s*[.．、]/);
    return match ? Number(match[1]) : null;
  }

  function compareByOrderThenDate(a, b) {
    var aOrder = extractOrderNumber(a.title || a.name);
    var bOrder = extractOrderNumber(b.title || b.name);

    if (aOrder !== null && bOrder !== null && aOrder !== bOrder) {
      return aOrder - bOrder;
    }
    if (aOrder !== null && bOrder === null) return -1;
    if (aOrder === null && bOrder !== null) return 1;

    var aDate = a.oldestDate || a.date || '';
    var bDate = b.oldestDate || b.date || '';
    if (aDate !== bDate) return aDate.localeCompare(bDate);

    return String(a.title || a.name).localeCompare(String(b.title || b.name), 'zh-Hans-CN');
  }

  function sortTree(node) {
    node.posts.sort(compareByOrderThenDate);
    node.children.sort(compareByOrderThenDate);
    node.children.forEach(sortTree);
  }

  function getChild(node, name) {
    if (!node.childMap[name]) {
      node.childMap[name] = makeNode(name);
      node.children.push(node.childMap[name]);
    }
    return node.childMap[name];
  }

  function buildTree(items) {
    var root = makeNode('All posts');
    items.forEach(function(post) {
      var node = root;
      node.count += 1;
      if (!node.oldestDate || post.date < node.oldestDate) node.oldestDate = post.date;
      post.folders.forEach(function(folder) {
        node = getChild(node, folder);
        node.count += 1;
        if (!node.oldestDate || post.date < node.oldestDate) node.oldestDate = post.date;
      });
      node.posts.push(post);
    });
    sortTree(root);
    return root;
  }

  function folderKey(path) {
    return path.join('/');
  }

  function folderLabel(path) {
    return path.length ? path[path.length - 1] : 'All posts';
  }

  function postsForPath(path) {
    var items = !path.length ? posts : posts.filter(function(post) {
      if (post.folders.length < path.length) return false;
      return path.every(function(folder, index) {
        return post.folders[index] === folder;
      });
    });
    return items.slice().sort(compareByOrderThenDate);
  }

  function setActive(path) {
    var key = folderKey(path);
    treeEl.querySelectorAll('.blog-tree-action').forEach(function(btn) {
      btn.classList.toggle('active', btn.dataset.folderKey === key);
    });
  }

  function showPosts(path) {
    var items = postsForPath(path);
    var label = folderLabel(path);
    titleEl.textContent = label;
    kickerEl.textContent = path.length ? path.join(' / ') : 'All posts';
    countEl.textContent = items.length + (items.length === 1 ? ' post' : ' posts');
    setActive(path);
    renderPosts(items);
  }

  function renderTreeNode(node, path, parent) {
    var key = folderKey(path);
    var row = document.createElement('div');
    row.className = 'blog-tree-row';

    var action = document.createElement('button');
    action.type = 'button';
    action.className = 'blog-tree-action';
    action.dataset.folderKey = key;
    action.textContent = folderLabel(path);
    action.addEventListener('click', function(event) {
      event.preventDefault();
      event.stopPropagation();
      showPosts(path);
    });

    var count = document.createElement('span');
    count.className = 'blog-tree-count';
    count.textContent = node.count;

    row.appendChild(action);
    row.appendChild(count);

    if (node.children.length) {
      var details = document.createElement('details');
      details.className = 'blog-tree-folder';
      details.open = path.length === 0;
      var summary = document.createElement('summary');
      summary.appendChild(row);
      details.appendChild(summary);
      var children = document.createElement('div');
      children.className = 'blog-tree-children';
      node.children.forEach(function(child) {
        renderTreeNode(child, path.concat(child.name), children);
      });
      details.appendChild(children);
      parent.appendChild(details);
    } else {
      parent.appendChild(row);
    }
  }

  function renderPosts(items) {
    listEl.innerHTML = '';
    emptyEl.classList.toggle('hidden', items.length !== 0);

    items.slice().sort(compareByOrderThenDate).forEach(function(post) {
      var article = document.createElement('article');
      article.className = 'group border border-surface-container-high bg-surface-container-lowest p-6 transition-colors duration-200 hover:border-neutral-900 dark:hover:border-neutral-50';

      if (post.folders.length) {
        var crumb = document.createElement('div');
        crumb.className = 'mb-2 flex flex-wrap items-center gap-1 text-xs font-bold uppercase tracking-[0.15em] text-on-surface-variant';
        post.folders.forEach(function(folder, index) {
          if (index > 0) {
            var sep = document.createElement('span');
            sep.textContent = '›';
            crumb.appendChild(sep);
          }
          var span = document.createElement('span');
          span.textContent = folder;
          crumb.appendChild(span);
        });
        article.appendChild(crumb);
      }

      var meta = document.createElement('div');
      meta.className = 'mb-3 flex flex-wrap items-center gap-3 text-xs font-bold uppercase tracking-[0.24em] text-on-surface-variant';
      [post.date, post.city, post.author].filter(Boolean).forEach(function(value) {
        var span = document.createElement('span');
        span.textContent = value;
        meta.appendChild(span);
      });
      article.appendChild(meta);

      var heading = document.createElement('h3');
      heading.className = 'text-2xl font-semibold text-primary';
      var link = document.createElement('a');
      link.href = post.url;
      link.className = 'inline-flex items-center gap-2';
      link.appendChild(document.createTextNode(post.title));
      var icon = document.createElement('span');
      icon.className = 'material-symbols-outlined text-base transition-transform duration-200 group-hover:translate-x-1';
      icon.textContent = 'north_east';
      link.appendChild(icon);
      heading.appendChild(link);
      article.appendChild(heading);

      listEl.appendChild(article);
    });
  }

  var tree = buildTree(posts);
  renderTreeNode(tree, [], treeEl);
  showPosts([]);

  if (searchInput) {
    searchInput.addEventListener('input', function() {
      var q = this.value.trim().toLowerCase();
      if (!q) {
        showPosts([]);
        return;
      }
      titleEl.textContent = 'Search results';
      kickerEl.textContent = q;
      treeEl.querySelectorAll('.blog-tree-action').forEach(function(btn) {
        btn.classList.remove('active');
      });
      var results = posts.filter(function(post) {
        return post.title.toLowerCase().indexOf(q) !== -1 ||
          post.folders.join('/').toLowerCase().indexOf(q) !== -1;
      }).sort(compareByOrderThenDate);
      countEl.textContent = results.length + (results.length === 1 ? ' post' : ' posts');
      renderPosts(results);
    });
  }
})();
</script>
