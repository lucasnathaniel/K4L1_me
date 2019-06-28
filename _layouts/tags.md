---
layout: default
---
<section id="tags">
  <div class="container py-3">
    <div class="row">
      <div class="col">
	<h2 class="text-primary"><i class="fas fa-tag"></i> {{ site.data.tags[page.tag].name }}</h2>

<div class="container py-3">
{% include tags.html %}
</div>

{% if site.tags[page.tag] %}
  {% for post in site.tags[page.tag] %}
    {% capture post_year %}{{ post.date | date: '%Y' }}{% endcapture %}
    {% if forloop.first %}
      <p><span>{{ post_year }}</span></p>
      <div>
    {% endif %}
    
    {% if forloop.first == false %}
      {% assign previous_index = forloop.index0 | minus: 1 %}
      {% capture previous_post_year %}{{ site.tags[page.tag][previous_index].date | date: '%Y' }}{% endcapture %}
      {% if post_year != previous_post_year %}
      </div>
      <p><span>{{ post_year }}</span></p>
      <div>
      {% endif %}
    {% endif %}

    <h3>
        <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
    </h3>
        <p >{{ post.content | strip_html | truncatewords:20}}</p>
    
    {% if forloop.last %}
      </div>
    {% endif %}
  {% endfor %}
{% else %}
  <p>There are no posts for this tag.</p>
{% endif %}
     </div>
    </div>
  </div>
</section>
