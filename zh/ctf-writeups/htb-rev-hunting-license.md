---
layout: writeup
lang: zh
title: "HackTheBox - Hunting License (逆向工程)"
description: "逆向一个通过 XOR 和字符串比较验证许可证密钥的 Linux 二进制文件，在 Ghidra 中静态分析验证逻辑提取密钥。"
platform: "HackTheBox"
category: "Reversing"
difficulty: "Easy"
date: 2026-03-01
permalink: /zh/ctf-writeups/htb-rev-hunting-license/
---

## 题目信息

**平台：** HackTheBox
**类别：** 逆向
**难度：** Easy

## 初步分析

```bash
file license
# ELF 64-bit LSB executable, not stripped

strings license | grep -i "password\|key\|wrong\|correct"
# WRONG
# Correct!
# Exiting...
```

运行程序要求输入密码，错误则打印 `WRONG` 退出。二进制未剥离符号，加载进 Ghidra 后函数名均完整可见。

## 静态分析

验证函数连续进行三次检查：

**第一关：** 直接 `strcmp` 与硬编码字符串比较。

```c
if (strcmp(input, "PasswordNumeroUno") != 0) { fail(); }
```

第一个密码直接拿到。

**第二关：** 第二个密码在二进制中以 `0x13` 为密钥进行 XOR 编码，从 Ghidra 提取字节后解码：

```python
encoded = [0x53, 0x63, 0x72, 0x65, 0x74, ...]
print(''.join(chr(b ^ 0x13) for b in encoded))
# -> "P4ssw0rdTw0"
```

**第三关：** 第三个密码以反转形式存储，取出后反转即可。

## Flag

```
HTB{l1c3ns3_4cquir3d-hunt1ng_t1m3!}
```

## 总结

- 单字节 XOR 极易逆向，密钥通常作为常量直接出现在编码数据旁边
- 未剥离符号的二进制保留了函数名，在 Ghidra 中导航更方便
- `strings` 是打开反汇编器前的快速初筛手段
