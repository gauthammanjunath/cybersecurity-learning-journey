# Module 03 — Reconnaissance & OSINT

**Dates covered:** 22 Apr – 27 Apr 2026  
**Phase:** Passive + Active Reconnaissance  
**Environment:** Kali Linux (legal targets: mit.edu for demo purposes)

---

## Overview

Reconnaissance is the first phase of a penetration test. The goal is to gather as much information about a target as possible **before** directly interacting with it.

```
Recon phases:
1. Passive OSINT    → no direct contact with target
2. Active recon     → direct interaction (DNS queries, port scans)
```

---

## Tools Used

### theHarvester — Email & Subdomain OSINT
```bash
theHarvester --help
theHarvester -d mit.edu                      # basic harvest
theHarvester -d mit.edu -b hackertarget      # use specific data source
theHarvester -d target.com -b google,bing,linkedin  # multiple sources
```

**Finds:** email addresses, subdomains, employee names, open ports, associated hosts from public sources (Google, Bing, LinkedIn, Shodan, etc.)

---

### Sublist3r — Subdomain Enumeration
```bash
apt install sublist3r
sublist3r --help
sublist3r -d mit.edu          # enumerate subdomains
sublist3r -d mit.edu -v       # verbose output
```

**Use case:** discover hidden subdomains that may run older/unpatched services

---

### DNS Enumeration

**nslookup:**
```bash
nslookup cirs.mit.edu         # basic lookup
nslookup -type=MX mit.edu     # mail server lookup
nslookup -type=NS mit.edu     # name server lookup
```

**dnsenum:**
```bash
dnsenum --help
dnsenum mit.edu                                    # basic
dnsenum --dnsserver 8.8.8.8 --enum --noreverse mit.edu  # use Google DNS, no reverse lookup
```

**dnsrecon:**
```bash
dnsrecon -d mit.edu           # standard recon
dnsrecon -d mit.edu -t axfr   # attempt zone transfer
```

**Key DNS records to look for:** A, AAAA, MX, NS, TXT, CNAME, PTR

---

### Google Dorking
```
# Operators
site:         limit results to specific domain
intitle:      search page titles
intext:       search page body text
filetype:     filter by file type
inurl:        search within URLs

# Example queries
site:facebook.com inurl:help intitle:hacked intext:password
site:digitaloceanspaces.com                 # find exposed S3-style buckets
filetype:pdf site:target.com confidential   # find confidential PDFs
```

---

### recon-ng — Modular OSINT Framework
```bash
recon-ng                        # launch framework
marketplace search all          # list all available modules
marketplace install all         # install all modules

# Workflow per module:
# 1) Locate: marketplace search <keyword>
# 2) Load:   modules load <module_name>
# 3) Config: options set SOURCE <target>
# 4) Run:    run
```

**Similar to Metasploit but for OSINT — modular, database-driven, tracks all gathered info.**

---

## Recon Methodology (Pentest Standard)

```
Target: example.com
│
├── Passive OSINT
│   ├── theHarvester → emails, subdomains
│   ├── Google dorks → exposed files, login pages
│   ├── LinkedIn → employee names, tech stack
│   └── Shodan → internet-exposed services
│
├── DNS Enumeration
│   ├── dnsenum / dnsrecon → subdomains, zone transfers
│   └── nslookup → specific record types
│
└── Active Recon (next phase → Nmap)
    └── Port scanning, service detection
```

---

## Key Takeaways

- Passive recon leaves **no trace** on target systems — always start here
- Subdomains often expose forgotten/unpatched services
- DNS zone transfers (AXFR) can dump entire DNS database if misconfigured
- Google dorks can find exposed credentials, config files, and admin panels
- recon-ng structures and stores all gathered intelligence automatically
