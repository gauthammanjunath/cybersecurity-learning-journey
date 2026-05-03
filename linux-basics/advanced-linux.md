# 🐧 Advanced Linux Commands (Cybersecurity Focus)

## 🎯 Objective

To understand advanced Linux commands used in system administration and cybersecurity operations.

---

## 🔍 Searching & Filtering

### grep

Search for patterns in files.

Example:
grep -i "kali" /etc/passwd

Options:

* -i → case insensitive
* -n → show line number
* -r → recursive search

---

### find

Search for files and directories.

Example:
find / -name "*.txt"

Useful filters:

* -type f → files
* -type d → directories
* -size +10M → files larger than 10MB
* -perm 777 → specific permissions

---

## 📦 Package Management

### apt

Used to install and manage packages.

Commands:
apt update
apt install <package>
apt remove <package>

---

## ⚙️ Process Management

### ps

View running processes.

Example:
ps aux

---

### kill

Terminate processes.

Examples:
kill <PID>
kill -9 <PID>

---

## 🔐 File Permissions

### chmod

Change file permissions.

Example:
chmod 755 file.txt

---

## 🌐 Networking Commands

### ip a

Displays IP address.

### ping

Check connectivity.

### netstat / ss

Check open ports and connections.

---

## 📊 System Monitoring

### top

Real-time system usage.

### free -h

Memory usage.

### uptime

System running time.

---

## 🧠 Key Takeaways

* Linux commands are essential for cybersecurity tasks
* Searching, monitoring, and managing processes are critical skills
* These commands are frequently used in penetration testing
