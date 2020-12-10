---
permalink: /tags/
title: Tags
---

<div>
{% capture tags %}{% for tag in site.tags %}{{ tag | first }}{% unless forloop.last %},{% endunless %}{% endfor %}{% endcapture %}
{% assign tag = tags | split:',' | sort %}
{% for item in (0..site.tags.size) %}{% unless forloop.last %}
{% capture word %}{{ tag[item] | strip_newlines }}{% endcapture %}
<h2 class="tag" id="{{ word }}">{{ word }}</h2>
<ul>
{% for post in site.tags[word] %}{% if post.title != null %}
<li><span>{{ post.date | date: "%b %d" }}</span>» <a href="{{ site.baseurl}}{{ post.url }}">{{ post.title }}</a></li>
{% endif %}{% endfor %}
</ul>
{% endunless %}{% endfor %}
<br/><br/>
</div>