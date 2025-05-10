---
layout: page
title: Tags
subtitle: Blader door berichten per tag
cover-img: /images/banner.jpg
---

{%- capture site_tags -%}
    {%- for tag in site.tags -%}
        {{- tag | first -}}{%- unless forloop.last -%},{%- endunless -%}
    {%- endfor -%}
{%- endcapture -%}
{% assign tags_list = site_tags | split:',' | sort %}

<div class="tags-expo">
  <div class="tags-expo-list">
    {% for tag in tags_list %}
    <a href="#{{ tag | slugify }}" class="post-tag">{{ tag }}</a>
    {% endfor %}
  </div>
  <hr/>
  <div class="tags-expo-section">
    {% for tag in tags_list %}
    <h2 id="{{ tag | slugify }}">{{ tag }}</h2>
    <ul class="tags-expo-posts">
      {% for post in site.tags[tag] %}
        <li>
          <a class="post-title" href="{{ site.baseurl }}{{ post.url }}">
            {{ post.title }}
          </a>
          <small class="post-date">Geplaatst op {{ post.date | date: "%-d" }} 
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
  </div>
</div>
