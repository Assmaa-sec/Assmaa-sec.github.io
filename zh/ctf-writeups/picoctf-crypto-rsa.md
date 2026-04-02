---
layout: writeup
lang: zh
title: "PicoCTF - Mind Your Ps and Qs (RSA 弱素数)"
description: "RSA 题目使用了过小的模数，直接用 sympy 分解 n，从头计算私钥还原明文。"
platform: "PicoCTF"
category: "Crypto"
difficulty: "Medium"
date: 2026-01-30
permalink: /zh/ctf-writeups/picoctf-crypto-rsa/
---

## 题目信息

**平台：** PicoCTF
**类别：** 密码学
**难度：** Medium

## 题目数据

```
c = 964354128913912393938480857590969826308054462950561875638492039
n = 16157656843214630540782260519598878842336783179741003288952602287996428169932265529
e = 65537
```

## 分析

首先注意到 `n` 只有约 59 位十进制数字。真实的 RSA 至少使用 2048 位密钥，对应 617 位以上的十进制数。模数如此之小，可以直接分解，不需要任何高级攻击手段。

## 解题过程

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

两个素数相近，Fermat 方法几乎瞬间完成分解，`sympy.factorint` 立即出结果。

## Flag

```
picoCTF{sma11_N_n0_g0od_23540368}
```

## 总结

- RSA 的安全性依赖 `n` 难以分解，200 位模数基本没有安全性可言
- 两个素数相近时，`sympy.factorint` 或 `factordb` 可以直接破解
- 密钥长度不是可选项，实际应用中至少 2048 位
