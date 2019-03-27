---
layout: page
title: "Archive"
description: "这里是从无到有之地"
header-img: "img/archive.jpg"
---

<style>
    .listing-seperator{
        list-style:none;
        font-weight:bold;
    }
</style>

<ul class="listing">
{% for post in site.posts %}
  {% capture y %}{{post.date | date:"%Y"}}{% endcapture %}
  {% if year != y %}
    {% assign year = y %}
    <li class="listing-seperator" id="{{ y }}">
    <a href="#top" title="{{ post.title }}">{{ y }}</a>
    </li>

  {% endif %}
  <li class="listing-item">
    <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>