---
layout: page
title: LearnOpenGL.com
---

This page contains the Polish translation of the most popular OpenGL tutorial series on the Interent - [learnopengl.com](http://learnopengl.com).

### Spis treści

<ul>
{% assign posts=site.posts | where:"subtag", 'intro-learnopengl' | sort: post.date %}
{% for post in posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}  


<details>
  <summary>Pierwsze kroki</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'pierwsze-kroki' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>

<details>
  <summary>Oświetlenie</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'oswietlenie' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>

<details>
  <summary>Zaawansowany OpenGL</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'zaawansowany-opengl' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>
</ul>