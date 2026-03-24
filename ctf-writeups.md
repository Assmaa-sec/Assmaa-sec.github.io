---
layout: page
title: CTF Writeups
subtitle: "$ grep -r 'flag{' ~/ctf/"
permalink: /ctf-writeups/
---

<div class="cards-grid">
  {% assign writeups = site.ctf_writeups | sort: "date" | reverse %}
  {% for writeup in writeups %}
  <div class="card">
    <div class="card-meta">
      <span class="card-tag">{{ writeup.category | default: "CTF" }}</span>
      {% if writeup.difficulty %}<span class="card-difficulty {{ writeup.difficulty | downcase }}">{{ writeup.difficulty }}</span>{% endif %}
      <span class="card-date">{{ writeup.date | date: "%b %d, %Y" }}</span>
    </div>
    <h3>{{ writeup.title }}</h3>
    {% if writeup.platform %}<p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">{{ writeup.platform }}{% if writeup.category %} · {{ writeup.category }}{% endif %}</p>{% endif %}
    <p>{{ writeup.description }}</p>
    <a href="{{ writeup.url | relative_url }}" class="card-link">Read writeup</a>
  </div>
  {% endfor %}
</div>

{% if site.ctf_writeups.size == 0 %}
<div style="color:#555; text-align:center; padding:4rem 0;">
  <p>// No writeups yet. Add files to <code>_ctf_writeups/</code> to get started.</p>
</div>
{% endif %}
