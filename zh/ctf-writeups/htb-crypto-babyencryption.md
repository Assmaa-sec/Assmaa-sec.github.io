---
layout: writeup
lang: zh
title: "HackTheBox - BabyEncryption (自定义线性密码还原)"
description: "逆向一个逐字节应用的自定义线性密码，通过模逆元还原加密函数，恢复明文。"
platform: "HackTheBox"
category: "Crypto"
difficulty: "Easy"
date: 2026-02-01
permalink: /zh/ctf-writeups/htb-crypto-babyencryption/
---

## 题目信息

**平台：** HackTheBox
**类别：** 密码学
**难度：** Easy

## 题目内容

题目提供了 `enc.py` 和输出文件，加密函数如下：

```python
def encryption(msg):
    ct = []
    for char in msg:
        ct.append(((123 * char + 18) % 256))
    return bytes(ct)
```

每个字节的变换公式为：`c = (123 * m + 18) % 256`

## 逆向推导

要还原原始字节，只需逆推公式：

```
c = (123 * m + 18) % 256
c - 18 = 123 * m  (mod 256)
m = (c - 18) * modular_inverse(123, 256)  (mod 256)
```

首先求 123 mod 256 的模逆元：

```python
inv = pow(123, -1, 256)  # 得到 179
```

## 解密脚本

```python
from Crypto.Util.number import long_to_bytes

ciphertext = bytes.fromhex("...<输出文件中的十六进制>...")

inv = pow(123, -1, 256)

plaintext = bytes([((c - 18) * inv) % 256 for c in ciphertext])
print(plaintext.decode())
```

## Flag

```
HTB{l00k_47_y0u_r3v3rs1ng_3qu4710n5_c0ngr475}
```

## 总结

- 形如 `c = (a*m + b) % n` 的密码，只要 `a` 和 `n` 互质，就可以直接逆推
- Python 3.8+ 内置 `pow(a, -1, n)` 支持模逆元计算，无需额外库
- 没有扩散性的自定义加密极其脆弱
