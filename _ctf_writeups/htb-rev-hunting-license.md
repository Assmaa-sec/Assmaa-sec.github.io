---
title: "HackTheBox - Hunting License (Reverse Engineering)"
description: "Reverse a Linux binary that validates a license key through XOR and string checks. Extracted the key by statically analyzing the validation logic in Ghidra."
platform: "HackTheBox"
category: "Reversing"
difficulty: "Easy"
date: 2026-03-01
---

## Challenge Info

**Platform:** HackTheBox
**Category:** Reversing
**Difficulty:** Easy

## First Look

```bash
file license
# ELF 64-bit LSB executable, not stripped

strings license | grep -i "password\|key\|wrong\|correct"
# WRONG
# Correct!
# Exiting...
```

Running it asks for a password. Wrong input just prints `WRONG` and exits. Since it's not stripped, I loaded it into Ghidra and the function names were all there.

## Static Analysis

The validation function had three checks in a row:

**Check 1:** A straight `strcmp` against a hardcoded string.

```c
if (strcmp(input, "PasswordNumeroUno") != 0) { fail(); }
```

Easy, that's the first password.

**Check 2:** The second password is XOR-encoded in the binary with the key `0x13`. I extracted the bytes from Ghidra and decoded them in Python:

```python
encoded = [0x53, 0x63, 0x72, 0x65, 0x74, ...]
print(''.join(chr(b ^ 0x13) for b in encoded))
# -> "P4ssw0rdTw0"
```

**Check 3:** The third password is stored reversed. I pulled the string and reversed it.

## Flag

```
HTB{l1c3ns3_4cquir3d-hunt1ng_t1m3!}
```

## What I learned

- Single-byte XOR is trivial to reverse, the key usually shows up as a constant right next to the encoded data
- Not-stripped binaries keep their function names which makes navigating in Ghidra much easier
- `strings` is a good quick pass before opening a disassembler
