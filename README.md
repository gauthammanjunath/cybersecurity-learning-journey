# Module 00 — Lab Environment Setup

**Dates:** 09–10 Mar 2026  
**Purpose:** Build isolated virtual lab for all penetration testing exercises

---

## Lab Architecture

```
Host Machine
│
└── VirtualBox 7.1.14
    ├── Kali Linux 2025.4 (attacker)
    ├── Metasploitable 2  (Linux target — intentionally vulnerable)
    ├── Windows 10        (Windows target)
    └── Custom CTF VMs    (HSBOX1, Straylight, Neuromancer, KGF1, KGF2)
```

---

## Network Modes Used

| Mode | Use case | IP Range |
|------|----------|----------|
| NAT Network | Isolated internet-like network between VMs | 10.0.2.0/24 |
| Host-Only Adapter | Isolated segment, no internet | 192.168.210.0/24 |
| Bridge Adapter | VM on same network as host | DHCP from router |

**Rule: Always disconnect VPN before running lab exercises.**

---

## Software Installed

### Windows Host
- VirtualBox 7.1.14
- VirtualBox Extension Pack
- Microsoft Visual C++ Redistributable
- 7-Zip (for extracting `.7z` VM images)

### Kali Linux (inside VM)
- Default Kali toolset (Nmap, Metasploit, Hydra, etc.)
- Nessus (vulnerability scanner)
- Sublist3r, recon-ng (installed during course)
- Ligolo-ng (pivoting tool)

---

## VM Credentials Reference

| VM | Username | Password |
|----|----------|----------|
| Kali Linux | kali | kali |
| Kali Linux (root) | root | (set during setup) |
| Metasploitable 2 | msfadmin | msfadmin |
| Parrot OS | root | toor |

---

## Common Setup Commands

```bash
# Set root password on Kali
sudo passwd root

# Check IP address
ip a

# Update package lists
apt update

# Install a .deb package
dpkg -i <package.deb>

# Start a service
service nessusd start
service ssh start
```

---

## Remote Support Tool
- **RustDesk** — used for instructor remote support during sessions
- Alternative to TeamViewer/AnyDesk for troubleshooting VM issues
