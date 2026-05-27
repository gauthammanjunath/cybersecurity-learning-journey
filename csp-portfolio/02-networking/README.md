# Module 02 — Networking Fundamentals

**Dates covered:** 30 Mar – 08 Apr 2026  
**Environment:** Kali Linux + Windows 10 on VirtualBox (NAT Network + Host-Only)

---

## Topics Covered

### IP Addressing & Subnetting

**IP Classes:**
| Class | Range | Default Subnet | Format |
|-------|-------|----------------|--------|
| A | 0–127 | 255.0.0.0 | N.H.H.H |
| B | 128–191 | 255.255.0.0 | N.N.H.H |
| C | 192–223 | 255.255.255.0 | N.N.N.H |

**Private IP Ranges (RFC 1918):**
```
Class A: 10.0.0.0      – 10.255.255.255
Class B: 172.16.0.0    – 172.31.255.255
Class C: 192.168.0.0   – 192.168.255.255
```

- Public IPs: routable on the internet, assigned by ISPs
- Private IPs: free to use internally, not routable externally (NAT required)

---

### Wireshark — Packet Analysis
```
- Promiscuous mode: captures ALL traffic on the network interface, not just traffic destined for your machine
- Used to: inspect packets, analyse protocols, detect credentials in cleartext
- Lab: captured traffic between Kali and Metasploitable2 on NAT Network
```

---

### Proxychains
```bash
# Config: /etc/proxychains4.conf

# Modes:
# dynamic  — skips dead proxies in chain (recommended)
# strict   — all proxies must be alive
# random   — random order through chain

proxychains nmap -sT <target>   # route nmap through proxy chain
```

**Use case:** anonymise traffic, route through Tor or SOCKS proxies

---

### iptables — Host Firewall

**IPv4:**
```bash
iptables -L                               # list all rules
iptables -A OUTPUT -p tcp --dport 80 -j DROP   # block outbound HTTP
iptables -D OUTPUT -p tcp --dport 80 -j DROP   # delete rule
iptables -A INPUT -p icmp -j ACCEPT       # allow ping in
iptables -A OUTPUT -p icmp -j ACCEPT      # allow ping out
```

**IPv6:**
```bash
ip6tables -L
ip6tables -A OUTPUT -p icmpv6 -j DROP     # block IPv6 ping
ip6tables -D OUTPUT -p icmpv6 -j DROP     # remove rule
```

**Direction reference:**
- `INPUT` = traffic coming into the machine
- `OUTPUT` = traffic leaving the machine
- `FORWARD` = traffic passing through (router scenario)

**Lab exercise:**
- Connected Kali + Windows 10 to same NAT network
- Created iptables rules to allow/block ICMP between machines
- Verified with `ping` before and after rule changes

---

### TCP Flags (for Nmap scan types)

| Flag | Meaning |
|------|---------|
| SYN | Synchronise — initiate connection |
| ACK | Acknowledge |
| FIN | Finish — close connection |
| RST | Reset — hard close |
| PSH | Push data immediately |
| URG | Urgent |
| NULL | No flags set |

---

### DNS Record Types

| Record | Purpose |
|--------|---------|
| A | Domain → IPv4 address |
| AAAA | Domain → IPv6 address |
| MX | Mail server for domain |
| NS | Authoritative name server |
| PTR | Reverse lookup (IP → domain) |

---

## Lab Environment Setup

```
Kali Linux ──────┐
                 ├── NAT Network (10.0.2.0/24)
Metasploitable2 ─┘

Kali Linux ──────────── Host-Only (192.168.210.0/24) ──── Windows 10
```

**Key lesson:** Always disconnect VPN before lab exercises to avoid routing issues.
