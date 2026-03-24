---
layout: home
---

<!-- ===== STATS ===== -->
<section class="section" style="padding-top: 3rem;">
  <div class="container">
    <div class="stats-bar">
      <div>
        <div class="stat-value">15+</div>
        <div class="stat-label">Projects</div>
      </div>
      <div>
        <div class="stat-value">50+</div>
        <div class="stat-label">CTF Challenges</div>
      </div>
      <div>
        <div class="stat-value">3+</div>
        <div class="stat-label">Years Exp.</div>
      </div>
      <div>
        <div class="stat-value">10+</div>
        <div class="stat-label">Certs &amp; Courses</div>
      </div>
    </div>
  </div>
</section>

<!-- ===== FEATURED PROJECTS ===== -->
<section class="section">
  <div class="container">
    <div class="section-header">
      <p class="section-label">Work</p>
      <h2 class="section-title">Featured Projects</h2>
      <div class="section-line"></div>
    </div>

    <div class="cards-grid">
      {% assign featured = site.projects | where: "featured", true | limit: 3 %}
      {% for project in featured %}
      <div class="card">
        <div class="card-meta">
          <span class="card-tag">{{ project.category | default: "Security" }}</span>
          {% if project.status %}<span class="card-tag">{{ project.status }}</span>{% endif %}
        </div>
        <h3>{{ project.title }}</h3>
        <p>{{ project.description }}</p>
        {% if project.tech %}
        <div class="tech-stack">
          {% for t in project.tech %}<span>{{ t }}</span>{% endfor %}
        </div>
        {% endif %}
        <a href="{{ project.url | relative_url }}" class="card-link">Read more</a>
      </div>
      {% endfor %}
    </div>

    <a href="{{ '/projects/' | relative_url }}" class="section-link">All Projects</a>
  </div>
</section>

<!-- ===== RECENT CTF WRITEUPS ===== -->
<section class="section">
  <div class="container">
    <div class="section-header">
      <p class="section-label">CTF</p>
      <h2 class="section-title">Recent Writeups</h2>
      <div class="section-line"></div>
    </div>

    <div class="cards-grid">
      {% assign writeups = site.ctf_writeups | sort: "date" | reverse | limit: 3 %}
      {% for writeup in writeups %}
      <div class="card">
        <div class="card-meta">
          <span class="card-tag">{{ writeup.category | default: "CTF" }}</span>
          {% if writeup.difficulty %}<span class="card-difficulty {{ writeup.difficulty | downcase }}">{{ writeup.difficulty }}</span>{% endif %}
          <span class="card-date">{{ writeup.date | date: "%b %Y" }}</span>
        </div>
        <h3>{{ writeup.title }}</h3>
        <p>{{ writeup.description }}</p>
        <a href="{{ writeup.url | relative_url }}" class="card-link">Read writeup</a>
      </div>
      {% endfor %}
    </div>

    <a href="{{ '/ctf-writeups/' | relative_url }}" class="section-link">All Writeups</a>
  </div>
</section>
