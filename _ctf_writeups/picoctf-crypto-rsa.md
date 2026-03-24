---
title: "PicoCTF — Mind Your Ps and Qs (RSA Weak Primes)"
description: "RSA challenge with an unusually small prime factor. Factored the modulus using Fermat's factorization algorithm and recovered the private key."
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
c = 1638...  (ciphertext)
n = 1615...  (modulus)
e = 65537    (public exponent)
```

## Analysis

The modulus `n` was suspiciously small — only 53 digits. Running it through Fermat's factorization confirmed one factor was close to √n.

## Solution

```python
from sympy import factorint
from Crypto.Util.number import long_to_bytes

n = 1615765684321463054078226051959887884233678317974...
e = 65537
c = 1638...

# Factor n
factors = factorint(n)
p, q = list(factors.keys())

# Recover private key
phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)          # modular inverse

# Decrypt
m = pow(c, d, n)
print(long_to_bytes(m))
```

## Output

```
b"picoCTF{sma11_N_n0_g0od_23f0e9b9}"
```

## Lessons

- RSA security depends entirely on `n` being hard to factor
- Small primes or primes close to each other break Fermat's factorization
- Always verify key sizes: `n` should be ≥ 2048 bits in production
