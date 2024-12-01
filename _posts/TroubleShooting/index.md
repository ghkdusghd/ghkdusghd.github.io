---
layout: page
title: TroubleShooting
---

{% for post in site.categories.projects %}

 <article>
   <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
   <p>{{ post.excerpt }}</p>
 </article>
{% endfor %}
