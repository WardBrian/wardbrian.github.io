---
title: "Programming Projects"
excerpt: "Personal and academic coding projects"
collection: portfolio
---


{% include base_path %}


{% for post in site.coding-portfolio reversed %}
{% include archive-single.html %}
{% endfor %}