# 🛡️ CyberOps SOC Home Lab — Red Team / Blue Team + IDS Detection

![SOC Lab](https://img.shields.io/badge/Type-SOC%20Home%20Lab-blue?style=for-the-badge)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-0078D4?style=for-the-badge)
![Snort](https://img.shields.io/badge/IDS-Snort-CC0000?style=for-the-badge)
![Kali Linux](https://img.shields.io/badge/Attacker-Kali%20Linux-557C94?style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/Defender-Ubuntu%20Server-E95420?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

> A fully functional **Red Team vs Blue Team** cybersecurity home lab simulating real-world attack scenarios, IDS-based network detection, and SOC monitoring workflows — built on a constrained hardware environment to reflect real-world resource limitations.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Lab Architecture](#lab-architecture)
- [Technology Stack](#technology-stack)
- [IDS — Snort Configuration](#ids--snort-configuration)
- [Attack Scenarios Simulated](#attack-scenarios-simulated)
- [Detection & Monitoring Workflow](#detection--monitoring-workflow)
- [Custom Snort Rules](#custom-snort-rules)
- [Wazuh + Snort Integration](#wazuh--snort-integration)
- [Incident Reporting Methodology](#incident-reporting-methodology)
- [MITRE ATT&CK Mapping](#mitre-attck-mapping)
- [Project Structure](#project-structure)
- [Learning Outcomes](#learning-outcomes)
- [Skills Demonstrated](#skills-demonstrated)
- [Disclaimer](#disclaimer)

---

## Overview

This project demonstrates a **production-style SOC (Security Operations Center) home lab** combining offensive security simulation with blue team detection and response capabilities.

The lab integrates:
- **Snort IDS** for real-time network intrusion detection
- **Wazuh SIEM** for log aggregation, alert correlation, and dashboard visibility
- **DVWA** as a vulnerable web application target
- **Kali Linux** as the attacker node running industry-standard offensive tools

Every attack simulated generates real network traffic, triggers IDS alerts, produces SIEM events, and is documented as a structured incident report — mirroring the workflow of a Tier 1 / Tier 2 SOC analyst.

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              VirtualBox Bridged Network                      │
│                   192.168.1.0/24                             │
│                                                              │
│   ┌──────────────┐        Attack Traffic        ┌─────────────────────────────┐  │
│   │  Kali Linux  │ ──────────────────────────► │       Ubuntu Server          │  │
│   │  (Attacker)  │                              │  ┌─────────┐  ┌──────────┐  │  │
│   │ 192.168.1.16 │                              │  │  Snort  │  │  Wazuh   │  │  │
│   └──────────────┘                              │  │   IDS   │  │   SIEM   │  │  │
│                                                 │  └────┬────┘  └────┬─────┘  │  │
│                                                 │       │  Alerts    │        │  │
│                                                 │       └────────────┘        │  │
│                                                 │         DVWA (Target)       │  │
│                                                 └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Attacker Node — Kali Linux**
- IP: `192.168.1.16`
- Role: Offensive security testing and attack simulation
- Tools: Nmap, Metasploit, SQLmap, Hydra, Nikto, Burp Suite

**Defender / SOC Node — Ubuntu Server**
- Role: Detection, monitoring, and incident analysis
- Services: Snort IDS, Wazuh SIEM, DVWA, Apache2, MySQL, UFW Firewall
- Interface: `enp0s3` (Bridged · `192.168.1.0/24`)

---

## Technology Stack

| Category | Tool | Purpose |
|---|---|---|
| Virtualization | VirtualBox | VM hosting and network simulation |
| Attacker OS | Kali Linux | Offensive security testing |
| Defender OS | Ubuntu Server | SOC monitoring node |
| IDS | Snort 2.9.20 | Network intrusion detection |
| SIEM | Wazuh | Log aggregation and alert correlation |
| Vulnerable App | DVWA | Web attack target |
| Firewall | UFW | Host-based firewall |
| Web Server | Apache2 | DVWA hosting |
| Database | MySQL / PHP | DVWA backend |

---

## IDS — Snort Configuration

Snort is deployed in **passive IDS mode** on the Ubuntu server, monitoring all traffic on `enp0s3`.

### Configuration highlights

- **HOME_NET:** `192.168.1.0/24`
- **Rule sets loaded:** Community rules + custom `local.rules`
- **Alert format:** `alert_fast` → `/var/log/snort/alert`
- **Total rules loaded:** 4,057 detection rules across 949 chain headers
- **Detection engine:** SF_SNORT_DETECTION_ENGINE v3.2

### Running Snort

```bash
# Start Snort in IDS mode
sudo snort -A fast -c /etc/snort/snort.conf -i enp0s3 -l /var/log/snort/

# Monitor alerts in real time
sudo tail -f /var/log/snort/alert

# Validate configuration
sudo snort -T -c /etc/snort/snort.conf -i enp0s3
```

---

## Custom Snort Rules

Custom detection rules written for lab-specific attack simulation:

```bash
# Detect Nmap SYN scan (T1046 — Network Service Discovery)
alert tcp $EXTERNAL_NET any -> $HOME_NET any \
  (msg:"NMAP SYN Scan detected"; flags:S; \
  threshold:type threshold, track by_src, count 20, seconds 3; \
  sid:1000001; rev:1;)

# Detect ICMP ping sweep (T1018 — Remote System Discovery)
alert icmp $EXTERNAL_NET any -> $HOME_NET any \
  (msg:"NMAP ICMP Ping Sweep"; itype:8; \
  threshold:type threshold, track by_src, count 5, seconds 3; \
  sid:1000002; rev:1;)

# Detect SSH brute force (T1110 — Brute Force)
alert tcp $EXTERNAL_NET any -> $HOME_NET 22 \
  (msg:"SSH Brute Force attempt"; flow:to_server; \
  threshold:type threshold, track by_src, count 5, seconds 10; \
  sid:1000003; rev:1;)

# Detect SQLmap activity (T1190 — Exploit Public-Facing Application)
alert tcp $EXTERNAL_NET any -> $HOME_NET 80 \
  (msg:"SQLmap SQL Injection attempt"; \
  flow:to_server,established; content:"sqlmap"; http_uri; \
  sid:1000004; rev:1;)
```

---

## Attack Scenarios Simulated

| Day | Attack | Tool | Snort SID | ATT&CK Technique |
|---|---|---|---|---|
| 01 | Network reconnaissance | Nmap | 1000001, 1000002 | T1046, T1018 |
| 02 | SSH brute force | Hydra | 1000003 | T1110.001 |
| 03 | SQL injection | SQLmap | 1000004 | T1190 |
| 04 | Web vulnerability scan | Nikto | Community rules | T1595.003 |
| 05 | Web exploitation | Burp Suite + DVWA | Community rules | T1059 |
| 06 | Service exploitation | Metasploit | Community rules | T1203 |

---

## Detection & Monitoring Workflow

```
1. Kali Linux launches attack against Ubuntu target
        │
        ▼
2. Snort IDS captures packets on enp0s3
   → Matches against 4,057 community + custom rules
   → Writes alert to /var/log/snort/alert
        │
        ▼
3. Wazuh manager reads Snort alert log
   → Parses using built-in snort-fast decoder
   → Correlates with other system logs
   → Generates SIEM alert with severity level
        │
        ▼
4. SOC analyst investigates in Wazuh dashboard
   → Reviews alert details, source IP, destination port
   → Pulls correlated events for full attack timeline
        │
        ▼
5. Incident report written with IOCs + ATT&CK mapping
```

---

## Wazuh + Snort Integration

Wazuh ingests Snort alerts directly via the `ossec.conf` localfile configuration:

```xml
<localfile>
  <log_format>snort-fast</log_format>
  <location>/var/log/snort/alert</location>
</localfile>
```

This enables:
- Automatic parsing of Snort fast-alert format
- Alert severity classification inside Wazuh
- Correlation of IDS events with system logs
- Unified dashboard visibility across all detection sources

---

## Incident Reporting Methodology

Each simulated attack produces a structured incident report following SOC analyst documentation standards:

```
incident-report-template/
├── Executive Summary
├── Timeline of Events
├── Attack Description
├── Tool Used
├── Target System
├── Indicators of Compromise (IOCs)
│   ├── Source IP
│   ├── Destination IP / Port
│   └── Snort SID triggered
├── Evidence
│   ├── Attack screenshots
│   ├── Snort alert output
│   └── Wazuh dashboard alerts
├── MITRE ATT&CK Technique
├── SOC Analyst Observations
├── Containment & Mitigation
└── Lessons Learned
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique Name | Tool Used | Detection Method |
|---|---|---|---|
| T1595.003 | Active Scanning | Nmap, Nikto | Snort SID 1000001-1000002 |
| T1110.001 | Password Spraying / Brute Force | Hydra | Snort SID 1000003 |
| T1190 | Exploit Public-Facing Application | SQLmap, Burp | Snort SID 1000004 |
| T1059 | Command and Scripting Interpreter | Metasploit | Wazuh + Community rules |
| T1018 | Remote System Discovery | Nmap | Snort SID 1000002 |
| T1046 | Network Service Discovery | Nmap | Snort SID 1000001 |

---

## Project Structure

```
Red-Blue-Team-Lab-Setup/
│
├── README.md
│
├── snort/
│   ├── local.rules              # Custom detection rules
│   └── snort-config-notes.md   # Configuration reference
│
├── wazuh/
│   └── ossec-snort-config.xml  # Wazuh Snort integration config
│
├── daily-incidents/
│   ├── day-01-reconnaissance/
│   │   ├── attack-screenshots/
│   │   ├── snort-alerts/
│   │   ├── wazuh-alerts/
│   │   └── incident-report.md
│   │
│   ├── day-02-bruteforce/
│   │   ├── attack-screenshots/
│   │   ├── snort-alerts/
│   │   ├── wazuh-alerts/
│   │   └── incident-report.md
│   │
│   ├── day-03-sql-injection/
│   │   ├── attack-screenshots/
│   │   ├── snort-alerts/
│   │   ├── wazuh-alerts/
│   │   └── incident-report.md
│   │
│   └── day-04-web-exploitation/
│       ├── attack-screenshots/
│       ├── snort-alerts/
│       ├── wazuh-alerts/
│       └── incident-report.md
│
└── architecture/
    └── lab-diagram.png
```

---

## Learning Outcomes

This project develops hands-on proficiency in:

| Skill Area | Specifics |
|---|---|
| Network Intrusion Detection | Snort rule writing, signature tuning, alert analysis |
| SIEM Operations | Wazuh log ingestion, alert correlation, dashboard analysis |
| Threat Detection | IOC identification, true/false positive triage |
| Incident Response | Structured reporting, timeline reconstruction |
| Offensive Security | Attack simulation using industry-standard tools |
| MITRE ATT&CK | Mapping real attacks to framework techniques |
| Linux Administration | Service configuration, log management, firewall rules |
| Network Analysis | Packet inspection, traffic monitoring, protocol analysis |

---

## Skills Demonstrated

```
SOC Analysis          ████████████████████  Expert (Hands-on)
Snort IDS             ████████████████░░░░  Advanced
Wazuh SIEM            ████████████████░░░░  Advanced
Incident Reporting    ████████████████████  Expert (Documented)
MITRE ATT&CK          ███████████████░░░░░  Intermediate-Advanced
Network Forensics     █████████████░░░░░░░  Intermediate
Penetration Testing   ████████████░░░░░░░░  Intermediate
```

---

## Roadmap

- [x] Red Team / Blue Team base lab (Kali + Ubuntu + Wazuh + DVWA)
- [x] Snort IDS deployment and configuration
- [x] Custom detection rule development
- [x] Wazuh + Snort SIEM integration
- [x] Structured incident reporting workflow
- [x] MITRE ATT&CK technique mapping
- [ ] Windows 10 endpoint + Sysmon telemetry (planned — hardware upgrade pending)
- [ ] Phishing detection lab (GoPhish + YARA)
- [ ] Threat hunting with Elastic Stack (ELK + Sigma rules)
- [ ] DFIR workstation (Autopsy + Volatility)

---

## Disclaimer

> This project is strictly for **educational purposes**. All attack simulations are performed within a controlled, isolated virtual lab environment.
> No testing was performed against external, production, or unauthorized systems.
> All tools are used in compliance with their respective terms of use.

---

## Author

**Gade Kuldeep**

Cybersecurity enthusiast building practical SOC analyst and offensive security skills through hands-on home lab environments.

[![GitHub](https://img.shields.io/badge/GitHub-GadeKuldeep-181717?style=flat-square&logo=github)](https://github.com/gadekuldeep)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/gadekuldeep)

---

*Keywords: SOC Analyst | Home Lab | Snort IDS | Wazuh SIEM | Red Team Blue Team | Intrusion Detection System | Cybersecurity Portfolio | MITRE ATT&CK | Incident Response | Network Security Monitoring | Kali Linux | DVWA | Threat Detection | Security Operations | CTF | Penetration Testing | Log Analysis | Alert Triage*
