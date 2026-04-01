---
layout: post
title: "Linux Firewall with Netfilter and iptables"
category: seed-labs
tags: [firewall, iptables, netfilter, linux-kernel, conntrack]
date: december 5, 2025
---

## Overview

This lab explores how firewalls work in Linux at both the kernel level and the user space level. The main topics are packet interception using Netfilter hooks, writing loadable kernel modules to implement custom filtering logic, and using iptables to build practical firewall policies. The lab walks through stateless filtering, stateful connection tracking, NAT, and rate limiting. The goal was to go from understanding the theory of firewalls to actually implementing and testing them.

## Environment Setup

- Linux (SEED Lab VM, kernel 5.4.0-54-generic)
- Docker containers:
  - seed-router
  - Host A (external, 10.9.0.x subnet)
  - Host 1, 2, 3 (internal, 192.168.60.x subnet)
- C compiler and make (for kernel module compilation)
- iptables (user-space firewall rule management)
- Netfilter / conntrack (kernel-level packet filtering and state tracking)

## Attack Methodology

### Task 1.A - Hello World Kernel Module

The first task was to compile a simple kernel module and observe it being loaded and removed. This established the basic workflow for all subsequent kernel module tasks.

```bash
# Build the module
make -C /lib/modules/5.4.0-54-generic/build \
    M=/home/seed/firewall/Labsetup/Files/kernel_module modules

# Load the module (prints "Hello World" via dmesg)
sudo insmod hello.ko

# Confirm it loaded
lsmod | grep hello

# Remove the module (prints "Bye-by")
sudo rmmod hello

# Verify via dmesg
dmesg | tail -n 5
```

### Task 1.B - Stateless Packet Filtering with Netfilter

The `seedFilter` kernel module was built from the `packet_filter` directory:

```bash
cd packet_filter
make -C /lib/modules/5.4.0-54-generic/build \
    M=/home/seed/firewall/Labsetup/Files/packet_filter modules
```

The module registers two Netfilter hooks. `registerFilter` sets them up when the module loads and `removeFilter` cleans them up on removal:

```c
void registerFilter(void) {
    printk(KERN_INFO "Registering filters.\n");

    hook1.hook     = printInfo;
    hook1.hooknum  = NF_INET_LOCAL_OUT;
    hook1.pf       = PF_INET;
    hook1.priority = NF_IP_PRI_FIRST;
    nf_register_net_hook(&init_net, &hook1);

    hook2.hook     = printInfo;
    hook2.hooknum  = NF_INET_POST_ROUTING;
    hook2.pf       = PF_INET;
    hook2.priority = NF_IP_PRI_FIRST;
    nf_register_net_hook(&init_net, &hook2);

    hook3.hook     = printInfo;
    hook3.hooknum  = NF_INET_FORWARD;
    hook3.pf       = PF_INET;
    hook3.priority = NF_IP_PRI_FIRST;
    nf_register_net_hook(&init_net, &hook3);

    hook4.hook     = printInfo;
    hook4.hooknum  = NF_INET_LOCAL_IN;
    hook4.pf       = PF_INET;
    hook4.priority = NF_IP_PRI_FIRST;
    nf_register_net_hook(&init_net, &hook4);

    hook5.hook     = printInfo;
    /* hook5 hooknum set elsewhere in file */
    nf_register_net_hook(&init_net, &hook5);
}

void removeFilter(void) {
    printk(KERN_INFO "The filters are being removed.\n");
    nf_unregister_net_hook(&init_net, &hook1);
    nf_unregister_net_hook(&init_net, &hook2);
    nf_unregister_net_hook(&init_net, &hook3);
    nf_unregister_net_hook(&init_net, &hook4);
    nf_unregister_net_hook(&init_net, &hook5);
}

MODULE_LICENSE("GPL");
```

Loading the module and testing:

```bash
sudo insmod seedFilter.ko
lsmod | grep seedFilter

# DNS now blocked by POST_ROUTING hook dropping UDP to 8.8.8.8:53
dig @8.8.8.8 www.example.com    # fails

sudo rmmod seedFilter
dig @8.8.8.8 www.example.com    # works again
```

### Task 1.2 - Observing All Netfilter Hooks

The module was rebuilt with `printInfo` attached to all five Netfilter hooks. A continuous ping to baidu.com was run while dmesg was watched. The output showed which hooks fired for a simple ICMP ping, confirming only a subset of the hooks triggers for any given traffic path.

```bash
make clean
make -C /lib/modules/5.4.0-54-generic/build \
    M=/home/seed/firewall/Labsetup/Files/packet_filter modules

sudo insmod seedFilter.ko
ping www.baidu.com
dmesg | tail -n 20
```

### Task 1.3 - Blocking Ping and Telnet

`seedFilter.c` was modified to add two hook functions at `PRE_ROUTING`. `preventPing` drops ICMP echo-request packets from a specific source IP. `preventTelnet` drops TCP packets destined for port 23 from the same source:

```c
unsigned int preventPing(void *priv, struct sk_buff *skb,
                         const struct nf_hook_state *state)
{
    struct iphdr   *iph;
    struct icmphdr *icmph;

    iph   = ip_hdr(skb);
    icmph = icmp_hdr(skb);

    unsigned char *saddr = (unsigned char *)&iph->saddr;

    if (iph->protocol == IPPROTO_ICMP && icmph->type == ICMP_ECHO &&
        (int)saddr[0] == 10 && saddr[2] == 2 && saddr[3] == 6)
    {
        printk(KERN_INFO "Dropping ping packet\n");
        return NF_DROP;
    }
    return NF_ACCEPT;
}

unsigned int preventTelnet(void *priv, struct sk_buff *skb,
                           const struct nf_hook_state *state)
{
    struct iphdr  *iph;
    struct tcphdr *tcph;

    iph  = ip_hdr(skb);
    tcph = tcp_hdr(skb);
    /* drops TCP packets destined for port 23 from the same source */
    ...
}
```

After loading the updated module, pings and telnet attempts to 10.2.0.15 returned 100% packet loss / connection refused, while other hosts were unaffected.

### Task 2 - iptables Rules

**Task 2.A - Accept ping only, block everything else**

```bash
iptables -P FORWARD ACCEPT
iptables -A INPUT  -p icmp --icmp-type echo-request -j ACCEPT
iptables -A OUTPUT -p icmp --icmp-type echo-reply   -j ACCEPT
iptables -P OUTPUT DROP
iptables -P INPUT  DROP
```

Logging into container A confirmed ping worked but telnet did not.

**Task 2.B - Internal vs external host policy**

Internal hosts are in 192.168.60.x, external hosts are in 10.9.0.0/24. Rules on the router:

```bash
iptables -A FORWARD -i eth0 -p icmp --icmp-type echo-request -j DROP
iptables -A FORWARD -i eth1 -p icmp --icmp-type echo-request -j DROP
iptables -A FORWARD -i eth0 -p icmp --icmp-type echo-request -j ACCEPT
iptables -P FORWARD DROP
```

Results:
- Internal host can ping external host.
- External host cannot ping internal host.
- Neither side can telnet to the other.

**Task 2.C - Selective connectivity to specific internal machines**

Rules on the router to allow A to reach only 192.168.60.5, and allow Host 1 to reach internal machines but not external:

```bash
iptables -A FORWARD -i eth0 -p tcp -d 192.168.60.5 --dport 23 -j ACCEPT
iptables -A FORWARD -i eth1 -p tcp -s 192.168.60.5 --sport 23 -j ACCEPT
iptables -P FORWARD DROP
```

Container A could telnet to 192.168.60.5 but not to other internal machines. Host 1 could reach other internal machines but not the external host 10.9.0.5.

### Task 3 - Connection Tracking (conntrack)

**Task 3.A - Observing conntrack state**

ICMP and UDP are connectionless but conntrack still tracks them using timeouts. TCP is tracked using explicit state transitions.

Netcat UDP listener for conntrack observation:

```bash
# On host
nc -lu 9090

# On client
nc -u 192.168.60.5 9090
hello
```

`conntrack -L` was used to observe state entries appearing and disappearing:
- ICMP: a short-lived entry appears during a ping and disappears shortly after.
- UDP: an inactivity-based timeout entry is created when data is sent.
- TCP: full state transitions (NEW, ESTABLISHED, TIME_WAIT) tracked via flags.

**Task 3.B - Stateful firewall using conntrack**

These rules implement a stateful firewall that allows internal hosts to initiate outbound TCP, permits external telnet only to one designated server, automatically accepts return traffic for established connections, and drops all other forwarded TCP:

```bash
iptables -F
iptables -P FORWARD ACCEPT
iptables -A FORWARD -i eth0 -p tcp -d 192.168.60.5 --dport 23 --syn \
    -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -i eth1 -p tcp --syn \
    -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -p tcp \
    -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -p tcp -j DROP
```

Test results:
- Host A: telnet to 192.168.60.5 succeeded, telnet to 192.168.60.6 failed.
- Host 1: could reach all internal machines, could not reach external host 10.9.0.5.

### Task 4 - Rate Limiting

Rules to limit how many packets from 10.9.0.5 are allowed into the internal network. The second DROP rule is required - without it, excess packets would still be accepted because the limit rule only matches allowed packets and does not automatically drop the rest.

```bash
iptables -F
iptables -P FORWARD ACCEPT
iptables -A FORWARD -s 10.9.0.5 -m limit --limit 10/minute --limit-burst 5 -j ACCEPT
iptables -A FORWARD -s 10.9.0.5 -j DROP
```

The first few pings went through at normal speed. After the 5th packet the rate limit kicked in and pings slowed noticeably (56.25% packet loss observed).

### Task 5 - NAT and Load Distribution

**Round-robin (nth mode)** - distributes every Nth packet to each host in rotation:

```bash
iptables -t nat -F
iptables -t nat -A PREROUTING -p udp --dport 8080 \
    -m statistic --mode nth --every 3 --packet 0 \
    -j DNAT --to-destination 192.168.60.5:8080
iptables -t nat -A PREROUTING -p udp --dport 8080 \
    -m statistic --mode nth --every 2 --packet 0 \
    -j DNAT --to-destination 192.168.60.6:8080
iptables -t nat -A PREROUTING -p udp --dport 8080 \
    -m statistic --mode nth --every 1 --packet 0 \
    -j DNAT --to-destination 192.168.60.7:8080
```

**Probability (random mode)** - sends a defined percentage of traffic to each host:

```bash
iptables -t nat -F
iptables -t nat -A PREROUTING -p udp --dport 80 \
    -m statistic --mode random --probability 0.1 \
    -j DNAT --to-destination 192.168.60.5:8080
iptables -t nat -A PREROUTING -p udp --dport 80 \
    -m statistic --mode random --probability 0.3 \
    -j DNAT --to-destination 192.168.60.6:8080
iptables -t nat -A PREROUTING -p udp --dport 80 \
    -m statistic --mode random --probability 0.6 \
    -j DNAT --to-destination 192.168.60.7:8080
```

A netcat UDP listener was started on each host:

```bash
nc -luk 8080
```

Many packets were sent from Host A:

```bash
echo hello | nc -u 10.9.0.11 8080
# repeated multiple times
```

The number of packets received by each host matched the configured probabilities. It was interesting to see that just by setting probability values in the NAT rules you can control exactly how much traffic each host receives.

## Key Findings

- Netfilter hooks give fine-grained control over packet processing at multiple points in the kernel's packet path.
- Stateless firewalls can block specific protocols and ports, but stateful firewalls using conntrack handle return traffic automatically and are more flexible.
- conntrack tracks even connectionless protocols like ICMP and UDP using timeouts, not just TCP.
- Rate limiting rules require an explicit DROP rule for excess traffic to actually work - the limit rule alone does not drop anything.
- NAT rules with `statistic` matching can distribute traffic across multiple hosts either in round-robin or by probability.

## Defense Implications

- Stateful firewall rules using conntrack reduce the chance of accidentally blocking legitimate return traffic.
- Default-deny policies (drop everything not explicitly allowed) are more secure than default-allow.
- Rate limiting is a straightforward way to reduce the impact of flooding from a known source IP.
- Combining Netfilter hooks with iptables gives defenders visibility and enforcement at both the kernel and user-space levels.
- Understanding conntrack helps when writing rules that need to correctly handle both connection-oriented and connectionless protocols.

## Tools Used

- Linux Netfilter (kernel-level hook framework)
- Loadable kernel modules (C, make, insmod, rmmod, lsmod, dmesg)
- iptables (user-space firewall rule management)
- conntrack (connection state tracking module)
- Docker (containerized network environment)
- netcat (connectivity testing and traffic verification)
- ping / telnet (network diagnostics)
