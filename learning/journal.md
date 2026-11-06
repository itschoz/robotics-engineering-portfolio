---
layout: page
title: Learning Journal
permalink: /learning/journal/
---

<div class="hub-wrap">
  <header class="page-intro"><div class="eyebrow">Notes from the workbench</div><h1>Learning Journal</h1><p>강의에서 배운 내용뿐 아니라 실제 실행한 명령, 관찰 결과, 오류와 해결 과정을 기록합니다.</p></header>
  <ul class="post-list">
    {% for post in site.posts %}
    <li><div class="journal-meta">{{ post.date | date: "%Y.%m.%d" }} · {{ post.categories | join: " / " }}</div><h3><a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a></h3><p>{{ post.excerpt | strip_html }}</p></li>
    {% endfor %}
  </ul>
</div>
