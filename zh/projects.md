---
layout: page
lang: zh
title: 项目
subtitle: "$ ls -la ~/projects/"
permalink: /zh/projects/
---

<div class="cards-grid">

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">AI 安全研究</span>
      <span class="card-tag">完成</span>
      <span class="card-date">2025年6月</span>
    </div>
    <h3>LLM 安全研究 - 本科毕业论文</h3>
    <p>针对大语言模型的三类攻击（成员推断、PII 泄露、对抗性越狱）和两类防御方法（金丝雀插入、SHAP 可解释性）的实证研究。成员推断 AUC 达 0.834，OpenLLaMA-3B 越狱成功率 100%。</p>
    <div class="tech-stack">
      <span>Python</span><span>PyTorch</span><span>GPT-2</span><span>LLaMA-3</span><span>OpenLLaMA-3B</span><span>SHAP</span><span>CUDA</span>
    </div>
    <div style="display:flex; gap:0.8rem; align-items:center; margin-top:0.5rem; flex-wrap:wrap;">
      <a href="{{ '/zh/projects/llm-security-thesis/' | relative_url }}" class="card-link">详情</a>
      <a href="https://github.com/Assmaa-sec/Thesis" class="card-link" target="_blank" rel="noopener">GitHub ↗</a>
    </div>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">应用密码学</span>
      <span class="card-tag">完成</span>
      <span class="card-date">2024年9月</span>
    </div>
    <h3>门限签名区块链钱包</h3>
    <p>3/5 门限 ECDSA 钱包，私钥在任何节点上均不以完整形式存在。采用分布式密钥生成（DKG）、Shamir 秘密共享和可验证秘密共享（VSS），通过 mTLS 传输和 SHA-256 哈希链审计日志保障安全。</p>
    <div class="tech-stack">
      <span>Python</span><span>Flask</span><span>Threshold ECDSA</span><span>Shamir SS</span><span>DKG / VSS</span><span>mTLS</span>
    </div>
    <div style="display:flex; gap:0.8rem; align-items:center; margin-top:0.5rem; flex-wrap:wrap;">
      <a href="{{ '/zh/projects/blockchain-wallet/' | relative_url }}" class="card-link">详情</a>
      <a href="https://github.com/Assmaa-sec/Blockchain-wallet" class="card-link" target="_blank" rel="noopener">GitHub ↗</a>
    </div>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">应用密码学</span>
      <span class="card-tag">完成</span>
      <span class="card-date">2024年6月</span>
    </div>
    <h3>本地密码管理器</h3>
    <p>基于 Python 和 Tkinter 构建的安全本地密码管理器。采用 Fernet 对称加密（AES-128-CBC + HMAC-SHA256）和 PBKDF2 密钥派生（480,000 次迭代），每条记录使用独立随机盐值，完全离线。</p>
    <div class="tech-stack">
      <span>Python</span><span>Tkinter</span><span>Fernet</span><span>PBKDF2HMAC</span><span>SQLite</span>
    </div>
    <div style="display:flex; gap:0.8rem; align-items:center; margin-top:0.5rem; flex-wrap:wrap;">
      <a href="{{ '/zh/projects/password-manager/' | relative_url }}" class="card-link">详情</a>
      <a href="https://github.com/Assmaa-sec/Password-manager-" class="card-link" target="_blank" rel="noopener">GitHub ↗</a>
    </div>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">蓝队防御</span>
      <span class="card-tag">完成</span>
      <span class="card-date">2024年3月</span>
    </div>
    <h3>Windows 日志异常检测器</h3>
    <p>分析 Windows 安全事件日志，自动标记暴力破解、非工作时间登录、权限提升、流量突增及新 IP 等安全异常，输出严重等级评分和结构化 JSON 报告。</p>
    <div class="tech-stack">
      <span>Python</span><span>Windows 事件日志</span><span>CLI</span>
    </div>
    <div style="display:flex; gap:0.8rem; align-items:center; margin-top:0.5rem; flex-wrap:wrap;">
      <a href="{{ '/zh/projects/log-anomaly-detector/' | relative_url }}" class="card-link">详情</a>
      <a href="https://github.com/Assmaa-sec/Log-anomaly-detector-windows" class="card-link" target="_blank" rel="noopener">GitHub ↗</a>
    </div>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">AI 安全</span>
      <span class="card-tag">完成</span>
      <span class="card-date">2024年1月</span>
    </div>
    <h3>LLM 提示词注入扫描器</h3>
    <p>Python 命令行工具，检测文本中的提示词注入攻击。识别角色覆盖、越狱尝试、数据泄露探针及间接注入标记，输出风险评分（0-100）和结构化 JSON 报告。</p>
    <div class="tech-stack">
      <span>Python</span><span>Regex</span><span>CLI</span>
    </div>
    <div style="display:flex; gap:0.8rem; align-items:center; margin-top:0.5rem; flex-wrap:wrap;">
      <a href="{{ '/zh/projects/llm-prompt-injection-scanner/' | relative_url }}" class="card-link">详情</a>
      <a href="https://github.com/Assmaa-sec/LLM_prompt_injection_scanner" class="card-link" target="_blank" rel="noopener">GitHub ↗</a>
    </div>
  </div>

</div>
