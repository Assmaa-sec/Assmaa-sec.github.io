---
layout: home
lang: zh
permalink: /zh/
---

<!-- ===== STATS ===== -->
<section class="section" style="padding-top: 3rem;">
  <div class="container">
    <div class="stats-bar">
      <div>
        <div class="stat-value">15+</div>
        <div class="stat-label">项目</div>
      </div>
      <div>
        <div class="stat-value">50+</div>
        <div class="stat-label">CTF 挑战</div>
      </div>
      <div>
        <div class="stat-value">3+</div>
        <div class="stat-label">年经验</div>
      </div>
      <div>
        <div class="stat-value">10+</div>
        <div class="stat-label">证书与课程</div>
      </div>
    </div>
  </div>
</section>

<!-- ===== FEATURED PROJECTS ===== -->
<section class="section">
  <div class="container">
    <div class="section-header">
      <p class="section-label">作品</p>
      <h2 class="section-title">精选项目</h2>
      <div class="section-line"></div>
    </div>

    <div class="cards-grid">

      <div class="card">
        <div class="card-meta">
          <span class="card-tag">AI 安全研究</span>
          <span class="card-tag">完成</span>
        </div>
        <h3>LLM 安全研究 - 本科毕业论文</h3>
        <p>针对大语言模型的三类攻击与两类防御方法的实证研究。成员推断 AUC 达 0.834，OpenLLaMA-3B 越狱成功率 100%。</p>
        <div class="tech-stack">
          <span>Python</span><span>PyTorch</span><span>GPT-2</span><span>LLaMA-3</span><span>SHAP</span>
        </div>
        <a href="{{ '/zh/projects/llm-security-thesis/' | relative_url }}" class="card-link">查看详情</a>
      </div>

      <div class="card">
        <div class="card-meta">
          <span class="card-tag">应用密码学</span>
          <span class="card-tag">完成</span>
        </div>
        <h3>门限签名区块链钱包</h3>
        <p>3/5 门限 ECDSA 钱包，私钥在任何节点上均不以完整形式存在。采用分布式密钥生成、Shamir 秘密共享和可验证秘密共享。</p>
        <div class="tech-stack">
          <span>Python</span><span>Flask</span><span>Threshold ECDSA</span><span>mTLS</span>
        </div>
        <a href="{{ '/zh/projects/blockchain-wallet/' | relative_url }}" class="card-link">查看详情</a>
      </div>

      <div class="card">
        <div class="card-meta">
          <span class="card-tag">AI 安全</span>
          <span class="card-tag">完成</span>
        </div>
        <h3>LLM 提示词注入扫描器</h3>
        <p>Python 命令行工具，检测文本中的提示词注入攻击，识别角色覆盖、越狱尝试和数据泄露探针，输出风险评分和 JSON 报告。</p>
        <div class="tech-stack">
          <span>Python</span><span>Regex</span><span>CLI</span>
        </div>
        <a href="{{ '/zh/projects/llm-prompt-injection-scanner/' | relative_url }}" class="card-link">查看详情</a>
      </div>

    </div>

    <a href="{{ '/zh/projects/' | relative_url }}" class="section-link">全部项目</a>
  </div>
</section>

<!-- ===== RECENT CTF WRITEUPS ===== -->
<section class="section">
  <div class="container">
    <div class="section-header">
      <p class="section-label">CTF</p>
      <h2 class="section-title">近期题解</h2>
      <div class="section-line"></div>
    </div>

    <div class="cards-grid">
      {% assign writeups = site.ctf_writeups | sort: "date" | reverse | limit: 3 %}
      {% for writeup in writeups %}
      <div class="card">
        <div class="card-meta">
          <span class="card-tag">{{ writeup.category | default: "CTF" }}</span>
          {% if writeup.difficulty %}<span class="card-difficulty {{ writeup.difficulty | downcase }}">{{ writeup.difficulty }}</span>{% endif %}
          <span class="card-date">{{ writeup.date | date: "%Y年%-m月" }}</span>
        </div>
        <h3>{{ writeup.title }}</h3>
        <p>{{ writeup.description }}</p>
        <a href="{{ writeup.url | relative_url }}" class="card-link">阅读题解</a>
      </div>
      {% endfor %}
    </div>

    <a href="{{ '/zh/ctf-writeups/' | relative_url }}" class="section-link">全部题解</a>
  </div>
</section>
