---
layout: default
title: Post Listing
---
<div class="mt3" align="center">
  <a href="#posts-by-date" class="button button-blue button-big">Posts by Date</a> <a href="#posts-by-tags" class="button button-blue button-big">Posts by Tags</a>
</div>
---
## Posts by Date
{% for post in site.posts %}
  {% assign currentdate = post.date | date: "%B %Y" %}
  {% if currentdate != date %}
<b>{{ currentdate }}</b>
    {% assign date = currentdate %} 
  {% endif %}
<ul>
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
</ul>
{% endfor %}

---

## Posts by Tags
{% for tag in site.tags %}
  {% assign t = tag | first %}
  {% assign posts = tag | last %}

<b>{{ t }}</b>
<ul>
{% for post in posts %}
  {% if post.tags contains t %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="date">{{ post.date | date: "%B %-d, %Y"  }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
{% endfor %}
