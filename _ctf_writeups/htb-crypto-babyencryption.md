---
title: "HackTheBox - BabyEncryption (Custom Cipher Reversal)"
description: "Reverse a custom linear cipher applied byte-by-byte. Recovered the plaintext by inverting the encryption function using modular arithmetic."
platform: "HackTheBox"
category: "Crypto"
difficulty: "Easy"
date: 2026-02-01
---

## Challenge Info

**Platform:** HackTheBox
**Category:** Cryptography
**Difficulty:** Easy

## Given

The challenge came with `enc.py` and the output. The encryption function looked like this:

```python
def encryption(msg):
    ct = []
    for char in msg:
        ct.append(((123 * char + 18) % 256))
    return bytes(ct)
```

Each byte gets transformed with: `c = (123 * m + 18) % 256`

## Reversing It

To get back the original byte, I just needed to reverse the formula:

```
c = (123 * m + 18) % 256
c - 18 = 123 * m  (mod 256)
m = (c - 18) * modular_inverse(123, 256)  (mod 256)
```

First find the modular inverse of 123 mod 256:

```python
inv = pow(123, -1, 256)  # gives 179
```

## Decryption Script

```python
from Crypto.Util.number import long_to_bytes

ciphertext = bytes.fromhex("...<hex from output file>...")

inv = pow(123, -1, 256)

plaintext = bytes([((c - 18) * inv) % 256 for c in ciphertext])
print(plaintext.decode())
```

## Flag

```
HTB{l00k_47_y0u_r3v3rs1ng_3qu4710n5_c0ngr475}
```

## What I learned

- Any cipher of the form `c = (a*m + b) % n` can be reversed as long as `a` and `n` are coprime
- Python 3.8+ has `pow(a, -1, n)` built in for modular inverses, no extra library needed
- Custom encryption like this with no diffusion is very weak
