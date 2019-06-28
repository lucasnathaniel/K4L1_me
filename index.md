---
layout: default
title: Home
---
<ul class="post-list">
  {% for post in site.posts limit:10 %}
    <li class="no-list-style">
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
      <h2>
      <a class="post-link no-underline" href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
      </h2>
      <p>{{ post.content | strip_html | truncatewords:40 }}</p>

      {% if post.tags.size > 0 %}
        {% capture tags_content %}{% if post.tags.size == 1 %}
        <span class="icon dripicons-tag">
        </span>

        {% else %}
        <span class="icon dripicons-tags">
        </span>
        {% endif %}{% endcapture %}

        {% for post_tag in post.tags %}
          {% assign tag = site.data.tags[post_tag] %}
          {% if tag %}{% capture tags_content_temp %}
            {{ tags_content }}
            <a class="post-content__tag small" href="/{{ post_tag }}/">{{ tag.name }}</a>
            {% if forloop.last == false %}, {% endif %}{% endcapture %}
            {% assign tags_content = tags_content_temp %}
          {% endif %}
        {% endfor %}
      {% else %}
        {% assign tags_content = '' %}
      {% endif %}
      <p class="post-meta">{{ tags_content }}</p>
    </li>
  {% endfor %}
</ul>
<div class="more-link">
  <p><a class="no-underline" href="/articles/">more &#10142;</a></p>
</div>

