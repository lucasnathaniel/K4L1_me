---
layout: default
title: Articles
permalink: /articles/
---

## Articles

{% include tags.html %}

{% for post in site.posts  %}
  {% capture this_year %}
    {{ post.date | date: "%Y" }}
  {% endcapture %}
  {% capture next_year %}
    {{ post.previous.date | date: "%Y" }}
  {% endcapture %}

  {% if forloop.first %}
  <span>{{ this_year }}</span>
  <ul>
  {% endif %}

    <li>
      <a href="{{ post.url | prepend: site.baseurl }}" class="post-article">{{ post.title }}</a>
    </li>

  {% if forloop.last %}
  </ul>
  {% else %}{% if this_year != next_year %}
  </ul>
  <span>{{ next_year }}</span>
  <ul>
  {% endif %}{% endif %}
{% endfor %}
