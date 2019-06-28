---
title: Team
layout: default
permalink: "/team/"
---

## Team

<div class="infos">
{% for member in site.data.members %}
<div id="info-img">
  <img src="{{ '/assets/images/members/' | relative_url }}{{ member.img }}">
</div>
<div id="info-team">
<ul>
  <li class="dripicons-user"> {{ member.name }}</li>
  <li class="dripicons-information"> {{ member.job }}</li>
  <li class="dripicons-checklist"> {{ member.skills }}</li>
  <li class="dripicons-mail"> <a href="mailto:{{ member.email }} " target="_blank">{{ member.email }}</a></li>
  {% if member.site %}
  <li class="dripicons-web"> <a href="{{ member.site }}" target="_blank">Site</a></li>
  {% endif %}
  {% if member.twitter %}
  <li class="dripicons-user-id"> <a href="https://twitter.com/{{ member.twitter }}" target="_blank">Twitter</a></li>
  {% endif %}
</ul>
</div>
{% if forloop.last == false %}
  <hr />
{% endif %}
{% endfor %}
</div>
