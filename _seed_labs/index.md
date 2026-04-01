---
layout: page
title: "SEED Labs"
category: seed-labs
---

## SEED Security Labs

These are writeups from the SEED Lab security exercises. Each lab covers a different attack or defense technique with hands-on implementation using Docker, Scapy, GDB, and Linux kernel tools.

| Lab | Description |
|-----|-------------|
| [ARP Cache Poisoning](arp-cache-poisoning) | Forging ARP packets to redirect LAN traffic and execute a man-in-the-middle attack between two victims using Scapy. |
| [Buffer Overflow Attack](buffer-overflow) | Exploiting unsafe `strcpy` in a Set-UID program to overwrite the return address and spawn a root shell using custom shellcode. |
| [ICMP Redirect Attack](icmp-redirect) | Sending forged ICMP redirect messages to manipulate a victim's routing cache and intercept traffic through a malicious router. |
| [Linux Firewall with Netfilter and iptables](firewall) | Building stateless and stateful firewalls using Netfilter kernel hooks, loadable kernel modules, conntrack, and iptables rules. |
