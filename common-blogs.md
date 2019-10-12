---
layout: default
---

<div class="posts">
  {% assign sorted-posts = site.posts | where: "category","common" %}
  {% for post in sorted-posts %}
    <article class="post">

      <h1 class="post-title"><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1>
      <div class="date">
        Written in {{ post.language }}, on {{ post.date | date: "%B %e, %Y" }}
      </div>

      <div class="entry">
        {{ post.excerpt | truncatewords:75 }}
      </div>

      <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More </a>
    </article>
  {% endfor %}
</div>
