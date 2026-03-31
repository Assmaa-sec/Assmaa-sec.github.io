---
title: "PicoCTF - Glory of the Garden (Hidden Data in Image)"
description: "Flag hidden in the raw bytes of a JPEG file. Standard image viewers don't show data appended after the end-of-image marker, but strings and xxd reveal it immediately."
platform: "PicoCTF"
category: "Forensics"
difficulty: "Easy"
date: 2026-02-08
---

## Challenge Info

**Platform:** PicoCTF
**Category:** Forensics
**Difficulty:** Easy

## Given

A single image file: `garden.jpg`. It opens fine and just looks like a photo of a garden.

## Solution

My first instinct with any image challenge is to run `strings` on it and look for anything readable:

```bash
strings garden.jpg | grep picoCTF
```

Output:

```
Here is a flag "picoCTF{more_than_m33ts_the_3y35a97d3bB}"
```

Done. The flag was appended in plaintext after the JPEG end-of-image marker (`FFD9`). Image viewers stop reading at that marker so they never display it, but it's still sitting there in the file.

You can also see it with `xxd`:

```bash
xxd garden.jpg | tail -20
```

## Flag

```
picoCTF{more_than_m33ts_the_3y35a97d3bB}
```

## What I learned

- `strings` should be the first thing you run on any binary in a forensics challenge
- JPEG files end at `FFD9`. Anything after that is hidden from viewers but readable with basic tools
- For harder steganography challenges, look into `steghide`, `zsteg`, and `exiftool`
