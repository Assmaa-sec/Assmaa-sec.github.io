---
layout: page
title: Projects
subtitle: "$ ls -la ~/projects/"
permalink: /projects/
---

<div class="cards-grid">
  {% assign sorted_projects = site.projects | sort: "date" | reverse %}
  {% for project in sorted_projects %}
  <div class="card">
    <div class="card-meta">
      <span class="card-tag">{{ project.category | default: "Security" }}</span>
      {% if project.status %}<span class="card-tag">{{ project.status }}</span>{% endif %}
      {% if project.date %}<span class="card-date">{{ project.date | date: "%b %Y" }}</span>{% endif %}
    </div>
    <h3>{{ project.title }}</h3>
    <p>{{ project.description }}</p>
    {% if project.tech %}
    <div class="tech-stack">
      {% for t in project.tech %}<span>{{ t }}</span>{% endfor %}
    </div>
    {% endif %}
    <div style="display:flex; gap:0.8rem; align-items:center; margin-top:0.5rem; flex-wrap:wrap;">
      <a href="{{ project.url | relative_url }}" class="card-link">Details</a>
      {% if project.github %}<a href="{{ project.github }}" class="card-link" target="_blank" rel="noopener">GitHub ↗</a>{% endif %}
    </div>
  </div>
  {% endfor %}
</div>

{% if site.projects.size == 0 %}
<div style="color:#555; text-align:center; padding:4rem 0;">
  <p>// No projects yet. Add files to <code>_projects/</code> to get started.</p>
</div>
{% endif %}
