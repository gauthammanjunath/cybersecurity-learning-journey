

---

## About This Repository

This repository documents my hands-on practical experience from the CSP course, including:

- Linux system administration and hardening
- Network fundamentals and packet analysis
- Reconnaissance and OSINT techniques
- Vulnerability scanning and assessment
- Exploitation using Metasploit and manual techniques
- Post-exploitation and pivoting
- CTF lab writeups (HSBOX1, Straylight, Neuromancer, KGF1/KGF2)

All labs were performed in isolated virtual environments (VirtualBox/Kali Linux + target machines). No real systems were targeted.

---

## Lab Environment

| Component | Tool |
|-----------|------|
| Attacker OS | Kali Linux 2025.4 |
| Virtualization | VirtualBox 7.1.14 |
| Target machines | Metasploitable 2, Windows 10, custom CTF VMs |
| Network setup | NAT Network + Host-Only Adapter (isolated) |
| Frameworks | Metasploit, Nmap NSE, Nessus, Hydra |

---

## Repository Structure

```
📁 00-setup/              → Lab environment setup notes
📁 01-linux-fundamentals/ → Linux commands, permissions, user management
📁 02-networking/         → IP addressing, subnetting, firewalls, Wireshark
📁 03-reconnaissance/     → OSINT, theHarvester, sublist3r, recon-ng, DNS enum
📁 04-scanning-enumeration/ → Nmap, NSE scripts, service fingerprinting
📁 05-vulnerability-assessment/ → Nessus, CVE analysis, CVSS scoring
📁 06-exploitation/       → Metasploit, EternalBlue, IRC backdoor, msfvenom
📁 07-post-exploitation/  → Pivoting with Ligolo, lateral movement
📁 08-ctf-labs/           → Full CTF writeups: HSBOX1, Straylight, Neuromancer, KGF
```

---

## Skills Demonstrated

**Reconnaissance:** OSINT · theHarvester · Sublist3r · recon-ng · dnsenum · dnsrecon · Google Dorks  
**Scanning:** Nmap (SYN, TCP, UDP, FIN, XMAS, NULL) · Service version detection · OS fingerprinting · NSE scripts  
**Vulnerability Assessment:** Nessus · CVE research · CVSS scoring · EPSS scoring  
**Exploitation:** Metasploit Framework · EternalBlue (MS17-010) · UnrealIRC backdoor · SSH brute force · FTP enumeration  
**Post-Exploitation:** Msfvenom payloads · Pivoting · Ligolo tunneling · Lateral movement  
**Networking:** TCP/IP · Subnetting · iptables · ip6tables · Wireshark · Proxychains  
**Linux:** File system · User management · Permissions · Partitioning · Package management (apt, dpkg, yum, rpm)

---

## CTF Labs Timeline

| Lab | Date | Topics |
|-----|------|--------|
| HSBOX1 | 29 Apr 2026 | Nmap, FTP enumeration, NSE scripts, SSH cracking |
| Metasploitable 2 | 30 Apr – 06 May 2026 | SSH version scan, EternalBlue, UnrealIRC backdoor |
| Client-side Exploits | 11–14 May 2026 | msfvenom payloads, staged/stageless |
| Pivoting lab | 18–19 May 2026 | Multi-network pivoting, routing |
| Straylight | 20–21 May 2026 | Custom CTF VM, port scan, privilege escalation |
| Neuromancer | 21 May 2026 | Chained CTF from Straylight |
| KGF1 + KGF2 | 25–26 May 2026 | NAT network, multi-machine, Ligolo pivoting |

---

## Certifications In Progress

- CompTIA Security+ SY0-701 *(exam target: September 2026)*

---

> All activities performed in legal, isolated lab environments for educational purposes only.
