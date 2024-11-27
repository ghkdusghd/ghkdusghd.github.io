---
layout: page
title: Projects
permalink: /projects/
---

{% for post in site.categories.projects %}

  <div class="post-preview">
    <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
    <p>{{ post.excerpt }}</p>
    <small>{{ post.date | date: "%B %d, %Y" }}</small>
  </div>
{% endfor %}
