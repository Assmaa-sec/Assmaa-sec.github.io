---
title: "NetRecon — Passive Recon Toolkit"
description: "Passive reconnaissance tool that aggregates OSINT from Shodan, Censys, certificate transparency logs, and DNS records to build a complete external attack surface map."
category: "Offensive"
status: "Complete"
featured: true
date: 2025-08-20
tech:
  - Python
  - Shodan API
  - Censys API
  - asyncio
  - Rich CLI
github: "https://github.com/yourusername/netrecon"
---

## Overview

NetRecon automates the external reconnaissance phase of a pentest engagement. Given a domain or IP range, it builds a comprehensive attack surface map without sending a single packet to the target.

## Data Sources

- **Shodan / Censys** — exposed ports, banners, TLS certs
- **Certificate Transparency** — subdomain enumeration via crt.sh
- **DNS** — passive DNS, MX, SPF, DMARC analysis
- **GitHub** — leaked credentials, API keys, internal paths

## Example Output

```
$ netrecon scan example.com

[+] Subdomains found: 47
[+] Exposed services: 12
    ├── 203.0.113.5:22  OpenSSH 7.4 (outdated)
    ├── 203.0.113.8:443 nginx/1.14.0 (EoL)
    └── 203.0.113.12:8080 Apache Tomcat/8.5.32
[!] Leaked API keys detected in public repo: 2
[!] Email server missing DMARC: p=none (spoofable)
```
