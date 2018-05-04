---
layout: page
title: "Archive"
description: "这里，从无到有"
header-img: "img/archive.jpg"
---

<style type="text/css">
    .archive-top{
        list-style:none;
        font-weight:bold;
    }
    .listing-seperator{
        list-style:none;
        font-weight:bold;
    }
</style>

<div id='tag_cloud' class="archive-top">
{% for post in site.posts %}
  {% capture y %}{{post.date | date:"%Y"}}{% endcapture %}
  <a href="#{{ y }}" title="{{ y }}" rel="{{ post[1].size }}">{{ y }}</a>
{% endfor %}
</div>

<ul class="listing">
{% for post in site.posts %}
  {% capture y %}{{post.date | date:"%Y"}}{% endcapture %}
  {% if year != y %}
    {% assign year = y %}
    <li class="listing-seperator">
    <a href="#top" title="{{ y }}">{{ y }}</a>
    </li>
  {% endif %}
  <li class="listing-item">
    <time datetime="{{ post.date | date:"%Y-%m-%d" }}">{{ post.date | date:"%Y-%m-%d" }}</time>
    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
  </li>
{% endfor %}
</ul>


<script src="/media/js/jquery.tagcloud.js" type="text/javascript" charset="utf-8"></script>
<script language="javascript">
$.fn.tagcloud.defaults = {
    size: {start: 1, end: 1, unit: 'em'},
      color: {start: '#f8e0e6', end: '#ff3333'}
};

$(function () {
    $('#tag_cloud a').tagcloud();
});
</script>