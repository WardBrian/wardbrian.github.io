---
title: "Programming Projects"
excerpt: "Personal and academic coding projects"
collection: portfolio
---


{% include base_path %}

*Note*: This page is still under construction!

{% for post in site.coding-portfolio reversed %}
{% include archive-single.html %}
{% endfor %}
