---
layout: page
title: Archief
subtitle: Volledige geschiedenis van blogberichten
cover-img: /images/banner.jpg
---

{% assign postsByYear = site.posts | group_by_exp:"post", "post.date | date: '%Y'" %}
{% for year in postsByYear %}
  <h2>{{ year.name }}</h2>
  <ul>
    {% for post in year.items %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <small class="text-muted">Geplaatst op {{ post.date | date: "%-d" }} 
        {% assign m = post.date | date: "%-m" %}
        {% case m %}
          {% when '1' %}januari
          {% when '2' %}februari
          {% when '3' %}maart
          {% when '4' %}april
          {% when '5' %}mei
          {% when '6' %}juni
          {% when '7' %}juli
          {% when '8' %}augustus
          {% when '9' %}september
          {% when '10' %}oktober
          {% when '11' %}november
          {% when '12' %}december
        {% endcase %}
        {{ post.date | date: "%Y" }}</small>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
