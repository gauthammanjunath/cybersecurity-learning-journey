# Module 01 — Linux Fundamentals

**Dates covered:** 11 Mar – 26 Mar 2026  
**Environment:** Kali Linux 2025.4 on VirtualBox

---

## Topics Covered

### User & Privilege Management
```bash
# Switch to root
sudo passwd root
sudo -i

# User structure in /etc/passwd
# Format: username:x:uid:gid:info:home:shell
cat /etc/passwd

# Password hashes stored in /etc/shadow
# Format: username:$hash:last_changed:...
cat /etc/shadow
```

**Key concepts:**
- `$` prompt = standard user
- `#` prompt = root/administrator
- UID 0 = root
- Passwords stored as hashed values in `/etc/shadow`

---

### File System Navigation
```bash
pwd           # print working directory
ls            # list contents
cd <dir>      # change directory
cd ..         # go up one level
cd            # go to home directory
mkdir <dir>   # create directory
mkdir -p a/b/c  # create nested directories
rmdir <dir>   # remove empty directory
rm -rf <dir>  # remove directory and all contents (use with caution)
find / -name <filename>  # search entire filesystem
```

**Practice exercises completed:**
1. Created `/home/new1/new2` nested directories
2. Created `/movies/2026/comedy`
3. Created directory with spaces: `kali linux` using quotes
4. Created `/var/www/html/Black hat`

---

### File Permissions
```bash
# Permission structure: [type][owner][group][other]
# Example: -rwxr-xr--
# r=read(4), w=write(2), x=execute(1)

chmod u+x file.txt    # add execute for owner
chmod g-w file.txt    # remove write for group
chmod o=r file.txt    # set other to read only
chmod 755 file.txt    # rwxr-xr-x (numeric)
chmod 644 file.txt    # rw-r--r-- (numeric)

# Symbols: u=owner, g=group, o=other, +=add, -=remove, ==exact
```

---

### Package Management

**Debian/Kali (apt/dpkg):**
```bash
apt update
apt install <package>
dpkg -i <package.deb>     # install local .deb
dpkg -r <package>         # remove package
```

**RedHat/CentOS (yum/rpm):**
```bash
yum install <package>
rpm -i <package.rpm>
```

---

### Disk & Storage
```bash
fdisk -l              # list all disks/partitions
fdisk /dev/sdb        # partition a disk interactively
df -h                 # disk space usage (human readable)
du -h /etc/passwd     # size of specific file
du -sh /etc           # total size of directory
```

**Lab exercise:** Added two virtual hard disks (4GB + 2GB) to Kali VM, partitioned and formatted them using `fdisk`.

---

### Networking Basics
```bash
ip a                  # show all network interfaces and IPs
hostname              # show hostname
ping <ip>             # test connectivity
```

---

## Key Takeaways

- Linux file permissions are foundational to privilege escalation understanding
- `/etc/passwd` and `/etc/shadow` are critical targets during post-exploitation
- Proper partitioning knowledge is important for forensic investigations
- Package management differs between Debian-based and RedHat-based distros
