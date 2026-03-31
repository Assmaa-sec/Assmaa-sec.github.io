---
title: "PicoCTF - Mind Your Ps and Qs (RSA Weak Primes)"
description: "RSA challenge with an unusually small modulus. Factored n directly using sympy and recovered the plaintext by computing the private key from scratch."
platform: "PicoCTF"
category: "Crypto"
difficulty: "Medium"
date: 2026-01-30
---

## Challenge Info

**Platform:** PicoCTF
**Category:** Cryptography
**Difficulty:** Medium

## Given

```
c = 964354128913912393938480857590969826308054462950561875638492039
n = 161576568432146305407822605195988788423367831797410032889526022879964281699322655 29
e = 65537
```

## Analysis

The first thing I noticed was how small `n` is, only around 59 digits. Real RSA uses at least 2048-bit keys which is 617+ digits. When `n` is this small you can just factor it directly, no fancy attack needed.

## Solution

```python
from sympy import factorint
from Crypto.Util.number import long_to_bytes

n = 16157656843214630540782260519598878842336783179741003288952602287996428169932265529
e = 65537
c = 964354128913912393938480857590969826308054462950561875638492039

factors = factorint(n)
p, q = list(factors.keys())

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)

print(long_to_bytes(m))
```

`sympy.factorint` cracked it almost instantly because the primes were close to each other, which makes Fermat's method trivial.

## Flag

```
picoCTF{sma11_N_n0_g0od_23540368}
```

## What I learned

- RSA only works if `n` is hard to factor. A 200-bit modulus has basically no security
- When primes are close together, `sympy.factorint` or `factordb` will break it right away
- Key size is not optional, 2048 bits minimum for anything real
