---
title: Team
layout: default
permalink: "/team/"
---

<section id="team">
  <div class="container py-3">
    <div class="row">
      <div class="col">
	<h2 class="text-primary">Team members</h2>

{% for member in site.data.members %}
{% assign loopindex = forloop.index | modulo: 2 %}
{% if loopindex == 1 %}
<div class="row">
 <div class="col py-1">
  <div class="media">
  <img class="mr-4" src="{{ '/assets/images/members/' | relative_url }}{{ member.img }}">
  <div class="media-body">
  <h5 class="mt-2"><i class="fas fa-user-secret"></i> {{ member.name }}</h5>
  <i class="far fa-question-circle"></i> {{ member.job }}<br />
  <i class="far fa-list-alt"></i> {{ member.skills }}<br />
  <i class="far fa-envelope"></i> <a href="mailto:{{ member.email }} " target="_blank">{{ member.email }}</a><br />
  {% if member.site %}
  <i class="fas fa-sitemap"></i> <a href="{{ member.site }}" target="_blank">Site</a><br />
  {% endif %}
  {% if member.twitter %}
  <i class="fab fa-twitter"></i> <a href="https://twitter.com/{{ member.twitter }}" target="_blank">Twitter</a><br />
  {% endif %}
  </div>
  </div>
  {% if forloop.last %}
  </div>
  {% endif %}
{% else %}
 <div class="col py-1">
  <div class="media">
  <img class="mr-4" src="{{ '/assets/images/members/' | relative_url }}{{ member.img }}">
  <div class="media-body">
  <h5 class="mt-2"><i class="fas fa-user-secret"></i> {{ member.name }}</h5>
  <i class="far fa-question-circle"></i> {{ member.job }}<br />
  <i class="far fa-list-alt"></i> {{ member.skills }}<br />
  <i class="far fa-envelope"></i> <a href="mailto:{{ member.email }} " target="_blank">{{ member.email }}</a><br />
  {% if member.site %}
  <i class="fas fa-sitemap"></i> <a href="{{ member.site }}" target="_blank">Site</a><br />
  {% endif %}
  {% if member.twitter %}
  <i class="fab fa-twitter"></i> <a href="https://twitter.com/{{ member.twitter }}" target="_blank">Twitter</a><br />
  {% endif %}
  </div>
  </div>
 </div>
{% endif %}
</div>
{% endfor %}
     </div>
    </div>
  </div>
</section>
