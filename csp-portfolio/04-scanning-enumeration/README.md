# Module 04 — Scanning & Enumeration

**Dates covered:** 13 Apr – 29 Apr 2026  
**Tools:** Nmap, NSE Scripts, Hydra  
**Target:** Metasploitable 2 (isolated lab — 10.0.2.15)

---

## Nmap — Network Mapper

### Host Discovery
```bash
nmap -sn 192.168.0.1               # ping scan single host
nmap -sn 192.168.0.0/24            # ping scan entire subnet
nmap -sn facebook.com              # ping scan by domain
nmap -6 -sn facebook.com           # IPv6 ping scan
nmap -sn 192.168.0.1 192.168.0.2 192.168.0.150   # multiple targets
```

---

### Scan Types

| Flag | Scan Type | Notes |
|------|-----------|-------|
| `-sT` | TCP Connect | Full 3-way handshake — noisy, no root needed |
| `-sS` | TCP SYN (Stealth) | Half-open scan — faster, needs root |
| `-sU` | UDP | Slow but finds DNS, SNMP, DHCP |
| `-sF` | FIN | Sends FIN flag — evades some firewalls |
| `-sX` | XMAS | Sets FIN+PSH+URG flags |
| `-sN` | NULL | No flags — evades some IDS |
| `-Pn` | Skip ping | Treat host as up even if no ping response |

```bash
# Examples
nmap -sT 192.168.0.43              # TCP connect scan
nmap -sS 192.168.0.43              # SYN scan (requires root)
nmap -sF 192.168.0.43              # FIN scan
nmap -sX 192.168.0.43              # XMAS scan
nmap -sN 192.168.0.43              # NULL scan
```

**FIN/XMAS/NULL scan logic:**
- Open port → no response
- Closed port → RST response
- Filtered → no response or ICMP unreachable

---

### Port Specification
```bash
nmap -sS 192.168.0.43 -p 21            # single port
nmap -sS 192.168.0.43 -p 21,22,80     # multiple ports
nmap -sS 192.168.0.43 -p 1-1000       # port range
nmap -sS 192.168.0.43 -p-             # all 65535 ports
nmap -sS 192.168.0.43 -F              # top 100 ports (fast)
nmap -sS 192.168.0.43 -dd             # top 1000 ports (default)
nmap -sS 192.168.0.43 -p ssh          # scan by service name
nmap -sT -sU -p T:21,22,80,U:53,67 192.168.0.43   # TCP + UDP combo
```

---

### Service & OS Detection
```bash
nmap -sT 192.168.0.43 -sV             # version detection
nmap -sT 192.168.0.43 -sV -O          # version + OS fingerprint
nmap -sS 192.168.0.43 -n -Pn -T4 -sV -O   # full stealth scan
```

**Timing templates (-T):**
- `-T0` paranoid (very slow), `-T1` sneaky, `-T2` polite
- `-T3` normal (default), `-T4` aggressive, `-T5` insane

---

### Nmap Scripting Engine (NSE)
```bash
find / -name *.nse                    # find all NSE scripts
find / -name *.nse | grep ftp         # find FTP-related scripts

# Get help on a specific script
nmap --script-help /usr/share/nmap/scripts/ftp-anon.nse

# Run a specific script
nmap -sS -T4 192.168.0.13 -sV -p 21 --script /usr/share/nmap/scripts/ftp-anon.nse

# FTP scripts used in lab
nmap ... --script ftp-anon.nse           # check for anonymous FTP login
nmap ... --script ftp-proftpd-backdoor.nse   # ProFTPd backdoor check
nmap ... --script ftp-vsftpd-backdoor.nse    # vsFTPd backdoor check

# SMB vulnerability scripts
nmap 10.0.2.20 -Pn -n -T4 -p 139,445 -sV --script smb-vuln-*
```

---

### CVE & Vulnerability Research
```bash
# CVE-2025-26674 — researched during class
# EPSS (Exploit Prediction Scoring System): 0.0 to 1.0 (probability of exploitation)
# CVSS (Common Vulnerability Scoring System): 0 to 10 (severity)
```

| Score System | Range | Purpose |
|---|---|---|
| CVSS | 0–10 | Severity of vulnerability |
| EPSS | 0%–100% | Probability it will be exploited in the wild |

---

### Hydra — Brute Force

```bash
hydra -h                              # help

# FTP brute force
hydra -L /root/username.txt -P /root/password.txt 192.168.0.13 -s 21 ftp -V
hydra -L /root/username.txt -P /root/password.txt 192.168.0.13 -s 21 ftp -t 2 -V   # 2 threads

# SMB brute force
hydra -L /root/username.txt -P /root/password.txt 192.168.0.13 -s 445 smb -V -t 25
hydra -L /root/username.txt -P /root/password.txt smb://192.168.0.53:445 -V -t 32
```

**Flags:**
- `-L` = username list file
- `-P` = password list file
- `-s` = custom port
- `-t` = number of parallel threads
- `-V` = verbose (show each attempt)

---

## Lab Results — Metasploitable 2 (10.0.2.15)

```
Open ports found: 21 (FTP), 22 (SSH), 23 (Telnet), 25 (SMTP),
                  80 (HTTP), 139/445 (SMB), 3306 (MySQL), 8080 (HTTP)

FTP: Anonymous login ENABLED (ftp-anon.nse confirmed)
SSH: Version OpenSSH 4.7p1 (outdated, multiple known CVEs)
SMB: Vulnerable to MS17-010 (EternalBlue) — confirmed by smb-vuln script
```

---

## Key Takeaways

- Always start with `-sn` host discovery before port scanning
- SYN scan (`-sS`) is the go-to — fast, stealthy, requires root
- NSE scripts transform Nmap from a scanner into a vulnerability finder
- Anonymous FTP and old SSH versions are extremely common in real environments
- CVSS tells you severity, EPSS tells you urgency — use both together
