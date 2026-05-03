# 🌐 DNS (Domain Name System)

## 🎯 Objective

To understand how domain names are translated into IP addresses and how DNS works in network communication.

---

## 🧠 What is DNS?

DNS (Domain Name System) converts human-readable domain names into IP addresses.

Example:
google.com → 142.250.74.14

---

## 🔄 How DNS Works

1. User enters a domain (e.g., google.com)
2. DNS query is sent to DNS server
3. DNS server resolves domain to IP address
4. Browser connects to the server using IP

---

## 📡 DNS Protocol

* Uses UDP (default)
* Port: 53

---

## 📌 DNS Record Types

### A Record

* Maps domain → IPv4 address

### AAAA Record

* Maps domain → IPv6 address

### MX Record

* Mail server information

### NS Record

* Name server information

### PTR Record

* Reverse lookup (IP → domain)

---

## 🔐 DNS in Cybersecurity

* Used in reconnaissance (DNS enumeration)
* Attackers gather subdomains and infrastructure info
* Can be abused in attacks like DNS spoofing

---

## 🧪 Example Command

nslookup google.com

---

## 🧠 Key Takeaways

* DNS translates domain names to IP addresses
* Essential for web communication
* Important for both attackers and defenders
