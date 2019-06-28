---
title: Sponsors
layout: default
permalink: "/sponsors/"
---

<section id="sponsors">
  <div class="container py-3">
    <div class="row">
      <div class="col">
	<h2 class="text-primary">Sponsors</h2>

{% if site.data.sponsors %} Our sponsors: {% endif %}

<div class="container">
  <div class="row">
{% for sponsor in site.data.sponsors %}
<div class="col-sm">
  <a href="{{ sponsor.site }}" target="_blank" title="{{ sponsor.name }}">
    <img src="{{ '/assets/images/sponsors/' | relative_url }}{{ sponsor.logo }}">
  </a>
  </div>
{% endfor %}
  </div>
</div>

Be one of our sponsors{% if site.data.sponsors %} too{% endif %}, please <a href="mailto:{{ site.email }}" target="_blank">contact</a> us.

     </div>
    </div>
  </div>
</section>
