---
title: "Password Manager"
description: "Secure local password manager built with Python and Tkinter. Uses Fernet encryption (AES-128-CBC + HMAC-SHA256) with PBKDF2 key derivation (480,000 iterations) and per-entry random salts. Fully offline with no network access."
category: "Applied Cryptography"
status: "Complete"
featured: true
date: 2024-06-01
tech:
  - Python
  - Tkinter
  - Fernet (AES-128-CBC)
  - PBKDF2HMAC
  - SQLite
github: "https://github.com/Assmaa-sec/Password-manager-"
---

## Overview

Desktop password vault that keeps every stored credential encrypted at rest. The master password is derived into an encryption key at runtime and never written to disk. If it is forgotten, stored passwords cannot be recovered.

## Security Design

| Property | Detail |
|---|---|
| Encryption | Fernet (AES-128-CBC + HMAC-SHA256) |
| Key derivation | PBKDF2HMAC, SHA-256, 480,000 iterations (NIST 2023 recommendation) |
| Salt | 16 random bytes per entry via `os.urandom`, stored in `passwords.db` |
| Password generation | `secrets` module (OS CSPRNG), guaranteed character class coverage + Fisher-Yates shuffle |
| Master password | Derived at runtime only, never stored, logged, or transmitted |
| Storage | SQLite file (`passwords.db`), fully local |

## Features

- Generates cryptographically secure passwords (16-128 chars)
- Each vault entry has its own salt, so compromising one entry does not affect others
- GUI with ADD / VIEW / DELETE / ABOUT tabs
- Copy-to-clipboard without revealing the password on screen

## Usage

```bash
pip install -r requirements.txt
python password_manager.py
```

A master password dialog appears on startup. All stored passwords are encrypted under the key derived from that password.
