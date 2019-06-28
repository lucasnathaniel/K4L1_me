---
layout: default
title: Articles
permalink: /articles/
---

<section id="articles">
  <div class="container py-3">
    <div class="row">
      <div class="col">
	<h2 class="text-primary">Articles</h2>
<div class="container py-3">
  <div class="row">
    <div class="col-sm-1">
      <p id="logo-tags" class="text-secondary text-center"><i class="fas fa-tags"></i></p>
    </div>
    {% include tags.html %}
  </div>
</div>
<div class="container py-3">
  <div class="row">
    <div class="col-sm-1">
      <p id="logo-tags" class="text-secondary text-center"><i class="fas fa-user-secret"></i></p>
    </div>
    {% include authors.html %}
  </div>
</div>

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
      <span>{{ post.date | date: "%b %d, %Y" }}</span> -
      <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
    </li>

  {% if forloop.last %}
  </ul>
  {% else %}{% if this_year != next_year %}
  </ul>
  <span>{{ next_year }}</span>
  <ul>
  {% endif %}{% endif %}
{% endfor %}
     </div>
    </div>
  </div>
</section>
