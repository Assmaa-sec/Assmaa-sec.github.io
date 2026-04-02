---
layout: page
lang: zh
title: CTF 题解
subtitle: "$ grep -r 'flag{' ~/ctf/"
permalink: /zh/ctf-writeups/
---

<div class="cards-grid">

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Pwn</span>
      <span class="card-difficulty hard">Hard</span>
      <span class="card-date">2026年3月</span>
    </div>
    <h3>HackTheBox - Format (格式化字符串 + malloc_hook 覆写)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">HackTheBox · Pwn</p>
    <p>利用格式化字符串漏洞泄露 libc 基址，随后将 __malloc_hook 覆写为 one_gadget 获取 shell。</p>
    <a href="{{ '/zh/ctf-writeups/htb-pwn-format/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Reversing</span>
      <span class="card-difficulty easy">Easy</span>
      <span class="card-date">2026年3月</span>
    </div>
    <h3>HackTheBox - Hunting License (逆向工程)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">HackTheBox · Reversing</p>
    <p>逆向通过 XOR 和字符串比较验证许可证密钥的 Linux 二进制文件，在 Ghidra 中静态分析提取密钥。</p>
    <a href="{{ '/zh/ctf-writeups/htb-rev-hunting-license/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Forensics</span>
      <span class="card-difficulty easy">Easy</span>
      <span class="card-date">2026年2月</span>
    </div>
    <h3>PicoCTF - Glory of the Garden (图片隐藏数据)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">PicoCTF · Forensics</p>
    <p>Flag 隐藏在 JPEG 文件结束标记之后的原始字节中，strings 和 xxd 可以立即发现。</p>
    <a href="{{ '/zh/ctf-writeups/picoctf-forensics-garden/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Web</span>
      <span class="card-difficulty easy">Easy</span>
      <span class="card-date">2026年2月</span>
    </div>
    <h3>HackTheBox - Templated (SSTI to RCE)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">HackTheBox · Web</p>
    <p>Flask/Jinja2 服务端模板注入，通过属性访问链绕过过滤实现远程代码执行。</p>
    <a href="{{ '/zh/ctf-writeups/htb-web-injection/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Crypto</span>
      <span class="card-difficulty easy">Easy</span>
      <span class="card-date">2026年2月</span>
    </div>
    <h3>HackTheBox - BabyEncryption (自定义线性密码还原)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">HackTheBox · Crypto</p>
    <p>逆向自定义线性密码，通过模逆元还原加密函数恢复明文。</p>
    <a href="{{ '/zh/ctf-writeups/htb-crypto-babyencryption/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Pwn</span>
      <span class="card-difficulty medium">Medium</span>
      <span class="card-date">2026年2月</span>
    </div>
    <h3>PicoCTF - Buffer Overflow 1 (Ret2Win)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">PicoCTF · Pwn</p>
    <p>32 位二进制经典栈溢出，覆写返回地址重定向执行流到隐藏的 win 函数。</p>
    <a href="{{ '/zh/ctf-writeups/picoctf-pwn-bof1/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Crypto</span>
      <span class="card-difficulty medium">Medium</span>
      <span class="card-date">2026年1月</span>
    </div>
    <h3>PicoCTF - Mind Your Ps and Qs (RSA 弱素数)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">PicoCTF · Crypto</p>
    <p>RSA 模数过小，直接用 sympy 分解 n，从头计算私钥还原明文。</p>
    <a href="{{ '/zh/ctf-writeups/picoctf-crypto-rsa/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Web</span>
      <span class="card-difficulty medium">Medium</span>
      <span class="card-date">2026年1月</span>
    </div>
    <h3>HackTheBox - Under Construction (JWT 算法混淆)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">HackTheBox · Web</p>
    <p>JWT 算法混淆攻击，以公钥作为 HMAC 密钥伪造 HS256 管理员令牌。</p>
    <a href="{{ '/zh/ctf-writeups/htb-web-under-construction/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

  <div class="card">
    <div class="card-meta">
      <span class="card-tag">Web</span>
      <span class="card-difficulty medium">Medium</span>
      <span class="card-date">2026年1月</span>
    </div>
    <h3>HackTheBox - Phonebook (LDAP 注入)</h3>
    <p style="font-size:0.72rem; color:#555; margin-bottom:0.4rem;">HackTheBox · Web</p>
    <p>LDAP 注入登录绕过，通配符实现任意用户认证，盲注枚举还原完整密码。</p>
    <a href="{{ '/zh/ctf-writeups/htb-web-phonebook/' | relative_url }}" class="card-link">阅读题解</a>
  </div>

</div>
