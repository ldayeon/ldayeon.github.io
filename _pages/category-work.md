---
title: "Work"
layout: archive
permalink: categories/work
author_profile: true
sidebar_main: true
class : wide
---
{% assign posts = site.categories.work %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}
