---
layout: project
lang: zh
title: "本地密码管理器"
description: "基于 Python 和 Tkinter 构建的安全本地密码管理器。采用 Fernet 对称加密（AES-128-CBC + HMAC-SHA256）和 PBKDF2 密钥派生（480,000 次迭代），每条记录使用独立随机盐值，完全离线。"
category: "应用密码学"
status: "完成"
featured: true
date: 2024-06-01
tech:
  - Python
  - Tkinter
  - Fernet (AES-128-CBC)
  - PBKDF2HMAC
  - SQLite
github: "https://github.com/Assmaa-sec/Password-manager-"
permalink: /zh/projects/password-manager/
---

## 项目概述

桌面端密码保险库，所有存储凭据均以加密形式保存。主密码在运行时派生为加密密钥，不写入磁盘。主密码一旦遗忘，已存储的密码将无法恢复。

## 安全设计

| 属性 | 详情 |
|---|---|
| 加密算法 | Fernet (AES-128-CBC + HMAC-SHA256) |
| 密钥派生 | PBKDF2HMAC，SHA-256，480,000 次迭代（NIST 2023 建议值） |
| 盐值 | 每条记录通过 `os.urandom` 生成 16 字节随机盐，存储于 `passwords.db` |
| 密码生成 | `secrets` 模块（操作系统 CSPRNG），保证字符类别覆盖 + Fisher-Yates 洗牌 |
| 主密码 | 仅在运行时派生，从不存储、记录或传输 |
| 存储 | SQLite 文件 (`passwords.db`)，完全本地 |

## 功能特性

- 生成密码学安全的密码（16-128 位）
- 每条记录使用独立盐值，单条记录泄露不影响其他记录
- GUI 界面包含添加 / 查看 / 删除 / 关于四个标签页
- 支持剪贴板复制，密码不在屏幕上明文显示

## 使用方式

```bash
pip install -r requirements.txt
python password_manager.py
```

启动时弹出主密码输入框。所有存储密码均使用从主密码派生的密钥加密。
