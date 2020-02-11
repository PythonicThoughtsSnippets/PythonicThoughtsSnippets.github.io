---
layout: default
---
{% include motivation.md %}
* * *
<h1>Latest Posts</h1>

<i>Click on the post title to read the full post.</i>

<ul>
  {% for post in site.posts %}
    <li>
      <h3><a href="{{ post.url }}">{{ post.date | date: '%d-%m-%Y' }} - {{ post.title }}</a></h3>
    </li>
  {% endfor %}
</ul>
