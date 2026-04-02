---
layout: page
lang: zh
title: CTF 题解
subtitle: "$ grep -r 'flag{' ~/ctf/"
permalink: /zh/ctf-writeups/
---

<div class="cards-grid">
  {% assign writeups = site.ctf_writeups | sort: "date" | reverse %}
  {% for writeup in writeups %}
  <div class="card">
    <div class="card-meta">
      <span class="card-tag">{{ writeup.category | default: "CTF" }}</span>
      {% if writeup.difficulty %}<span class="card-difficulty {{ writeup.difficulty | downcase }}">{{ writeup.difficulty }}</span>{% endif %}
      <span class="card-date">{{ writeup.date | date: "%Y年%-m月%-d日" }}</span>
    </div>
    <h3>{{ writeup.title }}</h3>
    {% if writeup.platform %}<p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">{{ writeup.platform }}{% if writeup.category %} · {{ writeup.category }}{% endif %}</p>{% endif %}
    <p>{{ writeup.description }}</p>
    <a href="{{ writeup.url | relative_url }}" class="card-link">阅读题解</a>
  </div>
  {% endfor %}
</div>

{% if site.ctf_writeups.size == 0 %}
<div style="color:#555; text-align:center; padding:4rem 0;">
  <p>// 暂无题解。</p>
</div>
{% endif %}
