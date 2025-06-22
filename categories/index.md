---
layout: page
title: "Category"
permalink: /categories/
---

<ul>
{% for category in site.categories %}
  <li>
    <a href="{{ site.baseurl }}/categories/{{ category[0] | downcase }}/">
  {{ category[0] }} ({{ category[1].size }})
</a>
  </li>
{% endfor %}
</ul>

