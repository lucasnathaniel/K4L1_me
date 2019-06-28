---
title: Sponsors
layout: default
permalink: "/sponsors/"
---

## Sponsors

Our sponsors:

<div class="sponsors-div">
{% for sponsor in site.data.sponsors %}
  <a class="sponsors-a" href="{{ sponsor.site }}" target="_blank" title="{{ sponsor.name }}">
    <img src="{{ '/assets/images/sponsors/' | relative_url }}{{ sponsor.logo }}" class="sponsors-img">
  </a>
{% endfor %}
</div>

Be one of our sponsors too, please <a href="mailto:contact@fireshellsecurity.team" target="_blank">contact</a> us.
