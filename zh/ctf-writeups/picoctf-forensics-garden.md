---
layout: writeup
lang: zh
title: "PicoCTF - Glory of the Garden (图片隐藏数据)"
description: "Flag 隐藏在 JPEG 文件的原始字节中，附加在文件结束标记之后。普通图片查看器看不到，但 strings 和 xxd 可以立即发现。"
platform: "PicoCTF"
category: "Forensics"
difficulty: "Easy"
date: 2026-02-08
permalink: /zh/ctf-writeups/picoctf-forensics-garden/
---

## 题目信息

**平台：** PicoCTF
**类别：** 取证
**难度：** Easy

## 题目内容

只有一个文件：`garden.jpg`，打开后就是一张普通的花园照片。

## 解题过程

拿到任何图片题，第一步就是跑 `strings` 看有没有可读内容：

```bash
strings garden.jpg | grep picoCTF
```

输出：

```
Here is a flag "picoCTF{more_than_m33ts_the_3y35a97d3bB}"
```

直接出了。Flag 以明文形式附加在 JPEG 文件结束标记（`FFD9`）之后。图片查看器在遇到结束标记时停止读取，所以不会显示，但数据确实在文件里，基础工具就能看到。

用 `xxd` 也能看到：

```bash
xxd garden.jpg | tail -20
```

## Flag

```
picoCTF{more_than_m33ts_the_3y35a97d3bB}
```

## 总结

- 取证题拿到二进制文件后，第一步跑 `strings`
- JPEG 以 `FFD9` 结束，之后附加的内容对查看器不可见，但对工具完全透明
- 更复杂的隐写题可以用 `steghide`、`zsteg`、`exiftool`
