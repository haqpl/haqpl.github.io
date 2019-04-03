---
layout: page
title: Photography
---

I like to take photos during walks. I will put a few here.

{% for repository in site.github.public_repositories %}
  * [{{ repository.name }}]({{ repository.html_url }})
{% endfor %}
