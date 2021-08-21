---
title: "Algorithm Basic"
layout: archive
permalink: categories/basic
author_profile: true
sidebar_main: true
class : wide
---
{% assign posts = site.categories.basic %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}
