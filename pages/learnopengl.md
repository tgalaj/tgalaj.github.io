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
  <summary style="margin-left: -20px;">Pierwsze kroki</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'pierwsze-kroki' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>

<details>
  <summary style="margin-left: -20px;">Oświetlenie</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'oswietlenie' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>

<details>
  <summary style="margin-left: -20px;">Ładowanie modeli</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'ladowanie-modeli' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>

<details>
  <summary style="margin-left: -20px;">Zaawansowany OpenGL</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'zaawansowany-opengl' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
  </ul>
</details>

<details>
  <summary style="margin-left: -20px;">Zaawansowane oświetlenie</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'advanced-lighting' | sort: post.date %}
  {% for post in posts %}
    {% assign post_date = post.date | date: '%d-%m-%Y' %}
    {% if post_date <= "03-10-2018" %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endif %}
  {% endfor %}

  <details>
    <summary style="margin-left: -20px;">Cienie</summary>
    <ul>
    {% assign posts=site.posts | where:"subtag", 'advanced-lighting-shadows' | sort: post.date %}
    {% for post in posts %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
    </ul>
  </details>  

  {% assign posts=site.posts | where:"subtag", 'advanced-lighting' | sort: post.date %}
  {% for post in posts %}
    {% assign post_date = post.date | date: '%d-%m-%Y' %}
    {% if post_date > "03-10-2018" %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endif %}
  {% endfor %}

  </ul>
</details>

<details>
  <summary style="margin-left: -20px;">PBR</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'pbr' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}

  <details>
    <summary style="margin-left: -20px;">IBL</summary>
    <ul>
    {% assign posts=site.posts | where:"subtag", 'pbr-ibl' | sort: post.date %}
    {% for post in posts %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
    </ul>
  </details>

  </ul>
</details>

<details>
  <summary style="margin-left: -20px;">W praktyce</summary>
  <ul>
  {% assign posts=site.posts | where:"subtag", 'in-practice' | sort: post.date %}
  {% for post in posts %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}

  <details>
    <summary style="margin-left: -20px;">Gra 2D</summary>
    <ul>
    {% assign posts=site.posts | where:"subtag", 'in-practice-2dgame' | sort: post.date %}
    {% for post in posts %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
    </ul>
  </details>

  </ul>
</details>

{% assign posts=site.posts | where:"subtag", 'intro-learnopengl-2' | sort: post.date %}
{% for post in posts %}
  <li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}  

</ul>