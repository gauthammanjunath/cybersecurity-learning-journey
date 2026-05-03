# 🌐 TCP/IP Model (Internet Model)

## 🎯 Objective

To understand the TCP/IP model and how real-world network communication works.

---

## 🧠 What is TCP/IP Model?

The TCP/IP model is the practical framework used on the internet for communication between devices.

---

## 🧱 4 Layers of TCP/IP Model

### 4️⃣ Application Layer

* Combines OSI layers: Application + Presentation + Session
* Protocols: HTTP, HTTPS, FTP, DNS

---

### 3️⃣ Transport Layer

* Handles data delivery and reliability
* Protocols:

  * TCP → Reliable (connection-based)
  * UDP → Fast (connectionless)

---

### 2️⃣ Internet Layer

* Handles IP addressing and routing
* Protocol: IP

---

### 1️⃣ Network Access Layer

* Handles physical transmission and MAC addressing
* Works with hardware (Ethernet, Wi-Fi)

---

## 🔄 Data Flow

Application → Transport → Internet → Network Access → Transmission

---

## 🔁 TCP/IP vs OSI Model

| TCP/IP Layer   | OSI Equivalent                       |
| -------------- | ------------------------------------ |
| Application    | Application + Presentation + Session |
| Transport      | Transport                            |
| Internet       | Network                              |
| Network Access | Data Link + Physical                 |

---

## 📌 Example

When you open a website:

* HTTP → Application Layer
* TCP → Transport Layer
* IP → Internet Layer
* MAC → Network Access Layer

---

## 🧠 Key Takeaways

* TCP/IP is used in real-world networking
* Simpler than OSI but very practical
* Essential for understanding tools like Wireshark and Nmap
