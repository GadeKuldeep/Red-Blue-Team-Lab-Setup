# 🛡️ CyberOps SOC Home Lab — Red Team vs Blue Team

<div align="center">

![SOC Lab](https://img.shields.io/badge/Type-SOC%20Home%20Lab-0d1117?style=for-the-badge&logo=shield&logoColor=00ff88)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-0078D4?style=for-the-badge&logo=elasticsearch&logoColor=white)
![Snort](https://img.shields.io/badge/IDS-Snort-CC0000?style=for-the-badge&logo=snort&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Red_Team-Kali_Linux-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Blue_Team-Ubuntu_Server-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![MITRE](https://img.shields.io/badge/Framework-MITRE_ATT%26CK-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-00ff88?style=for-the-badge)

**A production-style SOC home lab that simulates real-world cyberattacks and defends against them — end to end.**

*Attacker launches → IDS detects → SIEM correlates → Analyst responds → Incident documented.*

[View Incidents](#-incident-reports) · [See Detection Rules](#-custom-snort-rules) · [ATT&CK Mapping](#-mitre-attck-mapping)

</div>

---

## ⚡ What This Project Does — In 30 Seconds

```
🔴 Red Team (Kali Linux)          🔵 Blue Team (Ubuntu Server)
       │                                      │
  Launches real attacks              Snort IDS catches packets
  ─ Nmap recon                      Wazuh SIEM correlates alerts
  ─ Hydra SSH brute force    ──►    SOC Analyst investigates
  ─ SQLmap injection               Incident report written
  ─ Metasploit exploit             MITRE ATT&CK technique mapped
```

> This lab mirrors the **exact daily workflow of a Tier 1/Tier 2 SOC Analyst** — from the moment an attacker fires a scan, to detection, triage, and documented incident response.

---

## 🗺️ Lab Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    VirtualBox  ·  192.168.1.0/24                 │
│                                                                  │
│   ┌───────────────┐   attack traffic    ┌──────────────────────┐ │
│   │  Kali Linux   │ ──────────────────► │    Ubuntu Server     │ │
│   │  192.168.1.16 │                     │                      │ │
│   │               │                     │  ┌────────────────┐  │ │
│   │  Tools:       │                     │  │   Snort IDS    │  │ │
│   │  • Nmap       │                     │  │  (enp0s3 NIC)  │  │ │
│   │  • Hydra      │                     │  └───────┬────────┘  │ │
│   │  • SQLmap     │                     │          │ alerts    │ │
│   │  • Metasploit │                     │  ┌───────▼────────┐  │ │
│   │  • Nikto      │                     │  │  Wazuh SIEM   │  │ │
│   │  • Burp Suite │                     │  │  (dashboard)   │  │ │
│   └───────────────┘                     │  └────────────────┘  │ │
│                                         │                      │ │
│                                         │  Target: DVWA app    │ │
│                                         │  (Apache2 + MySQL)   │ │
│                                         └──────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

| Node | OS | IP | Role |
|---|---|---|---|
| 🔴 Attacker | Kali Linux | `192.168.1.16` | Red Team — launches all attacks |
| 🔵 Defender | Ubuntu Server | `192.168.1.14` | Blue Team — IDS + SIEM + SOC |
| 🎯 Target | DVWA (on Ubuntu) | `192.168.1.14:80` | Vulnerable web app under attack |

---

## 🔧 Why These Tools? — The "Why" Behind Every Choice

### 🔴 Snort IDS — *Because you need to catch attacks at the network layer*

Snort operates in **passive IDS mode**, sniffing all traffic on the network interface without blocking it. This mirrors how enterprise SOCs first detect threats — observe silently, then respond.

- Loaded with **4,057 community detection rules** covering known attack signatures
- Plus **6 custom rules** written from scratch for lab-specific scenarios
- Outputs alerts in `fast-alert` format to `/var/log/snort/alert`
- Rate-based thresholds tuned to minimize false positives

> **Why not a firewall instead?** A firewall *blocks* — but a SOC analyst needs to *see* and *understand* the attack first. Snort gives us full visibility.

---

### 🔵 Wazuh SIEM — *Because individual alerts mean nothing without correlation*

Wazuh ingests the Snort alert logs and correlates them with system-level events (auth logs, process logs, file changes) to give a **unified timeline** of what happened.

- Parses Snort's `fast-alert` format via built-in `snort-fast` decoder
- Assigns severity levels and groups related events
- Powers the SOC dashboard for analyst review
- Connects network-layer alerts (Snort) to host-layer telemetry

> **Why Wazuh over Splunk?** Wazuh is open-source, runs on modest hardware, and is widely used in real-world SOCs. It also has native Snort integration — perfect for a constrained lab environment.

---

### 🎯 DVWA — *Because you need a legal, realistic target*

DVWA (Damn Vulnerable Web Application) provides a web app with intentional flaws — SQL injection, XSS, brute-forceable login — giving the Red Team realistic targets without attacking real systems.

---

## 🔍 How Detection Works — Step by Step

```
Step 1  │  Kali launches attack (e.g., SQLmap against DVWA)
        │
Step 2  │  Snort captures packets on enp0s3
        │  → Matches against 4,057 community + 6 custom rules
        │  → Writes: [**] [1:1000004] SQLmap SQL Injection attempt [**]
        │
Step 3  │  Wazuh reads /var/log/snort/alert every few seconds
        │  → Parses alert using snort-fast decoder
        │  → Correlates with Apache2 access logs, auth logs
        │  → Generates SIEM event with severity + source IP
        │
Step 4  │  SOC Analyst opens Wazuh Dashboard
        │  → Reviews alert timeline, source IP, ports, payload hints
        │  → Identifies attack type and maps to MITRE ATT&CK
        │
Step 5  │  Incident Report written
        │  → IOCs documented, containment recommended
        │  → Lessons learned captured
```

---

## ✍️ Custom Snort Rules

All 6 rules written from scratch — no copy-paste from the internet:

```bash
# T1046 — Network Service Discovery: Nmap SYN scan
alert tcp $EXTERNAL_NET any -> $HOME_NET any \
  (msg:"NMAP SYN Scan detected"; flags:S; \
  threshold:type threshold, track by_src, count 20, seconds 3; \
  sid:1000001; rev:1;)

# T1018 — Remote System Discovery: ICMP ping sweep
alert icmp $EXTERNAL_NET any -> $HOME_NET any \
  (msg:"NMAP ICMP Ping Sweep"; itype:8; \
  threshold:type threshold, track by_src, count 5, seconds 3; \
  sid:1000002; rev:1;)

# T1110.001 — Brute Force: SSH password attack
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 \
  (msg:"SSH Brute Force attempt"; flow:to_server; \
  threshold:type threshold, track by_src, count 5, seconds 10; \
  sid:1000003; rev:1;)

# T1190 — Exploit Public-Facing App: SQLmap injection
alert tcp $EXTERNAL_NET any -> $HOME_NET 80 \
  (msg:"SQLmap SQL Injection attempt"; \
  flow:to_server,established; content:"sqlmap"; http_uri; \
  sid:1000004; rev:1;)
```

**Design decisions:**
- `track by_src` — rate limiting is per source IP, not global (avoids blocking legitimate traffic)
- `threshold` tuning — count/seconds values tested against real attack output to minimize false positives
- `flow:established` — confirms TCP session exists before matching, reduces noise

---

## ⚔️ Attack Scenarios Simulated

| # | Attack | Tool | ATT&CK Technique | Snort Rule Triggered | Alerts Generated |
|---|---|---|---|---|---|
| 1 | Network Reconnaissance | Nmap | T1046, T1018 | SID 1000001, 1000002 | ~3,873 in 30s |
| 2 | Automated Web Scanning | Nikto | T1595.003 | Community rules | ~7,238 in 12 min |
| 3 | SQL Injection | SQLmap | T1190 | SID 1000004 | Internal host |
| 4 | Multi-vector Attack | Nmap + Burp + Metasploit | T1059, T1203 | Multiple | ~9,276 in 1 min |
| 5 | Recon → Exploitation | Gobuster + SQLmap | T1190, T1595 | SID 1000001–4 | ~8,761 (authorized pen-test) |
| 6 | SSH Brute Force | Hydra | T1110.001 | SID 1000003 | Threshold-triggered |

---

## 📋 Incident Reports

Each attack produces a full structured incident report — the same format used in professional SOC environments:

```
incident-report-template/
├── 📌 Executive Summary          ← 2-line brief for management
├── ⏱️  Timeline of Events         ← Minute-by-minute reconstruction
├── 🔍 Attack Description          ← What happened technically
├── 🛠️  Tool Used                   ← Tool identified from signatures
├── 🎯 Target System               ← IP, port, service
├── 🚨 Indicators of Compromise    ← Source IP, ports, Snort SIDs
│   ├── Source IP
│   ├── Destination IP / Port
│   └── Snort SID triggered
├── 📸 Evidence                    ← Screenshots + log output
│   ├── Snort alert output
│   └── Wazuh dashboard view
├── 🗺️  MITRE ATT&CK Technique     ← Mapped technique ID + name
├── 🔵 SOC Analyst Observations    ← What the analyst concluded
├── 🛡️  Containment & Mitigation    ← Recommended actions
└── 📚 Lessons Learned             ← What this reveals about defenses
```

### Highlight — Incident 5 Plot Twist
> After detecting **8,761 alerts** showing a textbook Recon → SQL Injection progression from internal IP `192.168.1.15`, the SOC analyst escalated the incident — only to discover the source was an **authorized penetration tester** performing a scheduled engagement. Demonstrates real-world false positive / authorized activity investigation.

---

## 🗺️ MITRE ATT&CK Mapping

| Technique ID | Technique Name | Tool Used | Detected By |
|---|---|---|---|
| T1595.003 | Active Scanning | Nmap, Nikto | Snort SID 1000001–1000002 |
| T1110.001 | Password Spraying / Brute Force | Hydra | Snort SID 1000003 |
| T1190 | Exploit Public-Facing Application | SQLmap, Burp Suite | Snort SID 1000004 |
| T1059 | Command and Scripting Interpreter | Metasploit | Wazuh + community rules |
| T1018 | Remote System Discovery | Nmap | Snort SID 1000002 |
| T1046 | Network Service Discovery | Nmap | Snort SID 1000001 |

---

## 📁 Project Structure

```
Red-Blue-Team-Lab-Setup/
│
├── README.md                        ← You are here
├── PROJECT_SUMMARY.md               ← Executive overview
├── index.html                       ← Interactive network topology diagram
│
├── snort/
│   ├── local.rules                  ← 6 custom detection rules (written from scratch)
│   └── snort-config-notes.md        ← Configuration reference and tuning notes
│
├── wazuh/
│   └── ossec-snort-config.xml       ← Wazuh ↔ Snort integration config
│
├── daily-incidents/
│   ├── Incident_1/                  ← 3,873 alerts · Vulnerability scanning
│   │   ├── Report-1.txt
│   │   └── inhanced_report.txt
│   ├── Incident-2/                  ← 7,238 alerts · Automated web recon
│   │   ├── Incident-2_report.txt
│   │   └── enhanced_report.txt
│   ├── Incident-3/                  ← SQL Injection · Internal host
│   │   └── incident3_report.txt
│   ├── Incident-4/                  ← 9,276 alerts · Multi-vector attack
│   │   └── incident-4_report.txt
│   └── Incident-5/                  ← 8,761 alerts · Authorized pen-test reveal
│       ├── report_incident_5.md
│       └── report_ai.md             ← AI-assisted structured incident ticket
│
├── Snort-system/
│   ├── Day_1/report.md              ← Installation, HOME_NET config
│   └── Day_2/report.md             ← Custom ICMP rule testing + Wazuh integration
│
└── screenshots-ref/
    ├── snort_error/                 ← Troubleshooting screenshots
    └── *.jpg                        ← Dashboard views, alert outputs, configs
```

---

## 🧠 Skills Demonstrated

| Domain | What Was Done |
|---|---|
| **IDS Rule Writing** | 6 custom Snort signatures written from scratch with threshold tuning |
| **SIEM Operations** | Wazuh log ingestion, alert correlation, dashboard-based triage |
| **Incident Response** | 5 structured incident reports with IOCs, timelines, MITRE mapping |
| **Threat Detection** | True/false positive triage, attack tool fingerprinting |
| **Offensive Security** | Nmap, Hydra, SQLmap, Metasploit, Nikto, Burp Suite — live attack simulation |
| **Network Forensics** | HTTP error pattern analysis, automated tool identification from traffic |
| **Linux Administration** | Snort + Wazuh service config, UFW, Apache2, log management |
| **MITRE ATT&CK** | 6 techniques mapped across all incidents |

---

## 🗺️ Wazuh + Snort Integration

```xml
<!-- ossec.conf — tells Wazuh to ingest Snort alerts -->
<localfile>
  <log_format>snort-fast</log_format>
  <location>/var/log/snort/alert</location>
</localfile>
```

This single config block enables:
- Automatic parsing of Snort fast-alert format
- Severity classification inside Wazuh
- Cross-correlation with system auth and Apache logs
- Unified alert timeline on the Wazuh dashboard

---

## 🚀 Roadmap

- [x] Red Team / Blue Team base lab — Kali + Ubuntu + Wazuh + DVWA
- [x] Snort IDS deployment and passive monitoring configuration
- [x] 6 custom detection rules with threshold tuning
- [x] Wazuh + Snort SIEM integration
- [x] Structured incident reporting (5 incidents documented)
- [x] MITRE ATT&CK technique mapping across all incidents
- [ ] Windows 10 endpoint + Sysmon telemetry *(hardware upgrade pending)*
- [ ] Phishing detection lab — GoPhish + YARA rules
- [ ] Threat hunting with Elastic Stack — ELK + Sigma rules
- [ ] DFIR workstation — Autopsy + Volatility memory forensics

---

## 🚀 Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/GadeKuldeep/Red-Blue-Team-Lab-Setup.git

# 2. Start Snort in IDS mode on your defender node
sudo snort -A fast -c /etc/snort/snort.conf -i enp0s3 -l /var/log/snort/

# 3. Monitor alerts in real time
sudo tail -f /var/log/snort/alert

# 4. Validate Snort config before running
sudo snort -T -c /etc/snort/snort.conf -i enp0s3
```

> Full lab setup guide: see `Snort-system/Day_1/report.md` and `Snort-system/Day_2/report.md`

---

## ⚠️ Disclaimer

This project is strictly for **educational purposes**. All attack simulations are performed within a controlled, isolated virtual lab environment. No testing was performed against external, production, or unauthorized systems. All tools are used in compliance with their respective terms of use.

---

<div align="center">

**Built by Gade Kuldeep**

Cybersecurity student building practical SOC analyst skills through hands-on home lab environments.

[![GitHub](https://img.shields.io/badge/GitHub-GadeKuldeep-181717?style=flat-square&logo=github)](https://github.com/GadeKuldeep)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/kuldeep-gade-52598b2b0)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Profile-212C42?style=flat-square&logo=tryhackme)](https://tryhackme.com/p/gadekuldeep25)

</div>
