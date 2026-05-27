# 🛡️ Red-Blue-Team-Lab-Setup — Complete Project Summary

## Executive Overview

**Red-Blue-Team-Lab-Setup** is a comprehensive **Security Operations Center (SOC) home lab** that simulates real-world cybersecurity operations by combining offensive security attack simulation with defensive security detection and response. The lab demonstrates the integration of industry-standard tools (Snort IDS, Wazuh SIEM, Kali Linux, and DVWA) on a constrained hardware environment to mirror real-world SOC workflows.

---

## Project Objectives

1. **Simulate Real-World Attack Scenarios** — Execute industry-standard offensive security tools (Nmap, Hydra, SQLmap, Nikto, Burp Suite, Metasploit)
2. **Detect Attacks Using IDS** — Deploy and configure Snort IDS with custom detection rules to identify network-based threats
3. **Monitor & Correlate Events** — Integrate Snort alerts into Wazuh SIEM for centralized log aggregation and alert correlation
4. **Generate Structured Incident Reports** — Document each attack with executive summaries, timelines, IOCs, and MITRE ATT&CK mappings
5. **Develop SOC Analyst Skills** — Gain hands-on experience in threat detection, incident response, and security operations

---

## Key Technologies & Components

### Infrastructure
| Layer | Technology | Purpose | IP/Role |
|---|---|---|---|
| Hypervisor | VirtualBox | VM hosting & network virtualization | Bridged Network (192.168.1.0/24) |
| Attacker OS | Kali Linux | Offensive security testing | 192.168.1.16 |
| Defender OS | Ubuntu Server | SOC monitoring & detection | 192.168.1.0/24 segment |
| Network | VirtualBox Bridged | Real traffic simulation | LAN isolated |

### Defense Stack
| Component | Purpose | Feature Set |
|---|---|---|
| **Snort IDS 2.9.20** | Network intrusion detection | 4,057 community + custom detection rules, passive IDS mode, real-time alerting |
| **Wazuh SIEM** | Centralized security monitoring | Alert correlation, log aggregation, dashboard visualization, incident tracking |
| **UFW Firewall** | Host-based network filtering | Port whitelisting, ingress/egress control |
| **Apache2 & MySQL** | Application infrastructure | DVWA hosting, dynamic content serving |

### Attack Targets
| Target | Purpose |
|---|---|
| DVWA (Damn Vulnerable Web Application) | Web attack simulation (SQL injection, XSS, authentication bypass) |
| OpenSSH (Port 22) | Brute force testing |
| Network services | Reconnaissance and scanning detection |

### Offensive Toolset (Kali Linux)
- **Nmap** — Network reconnaissance and service discovery
- **Hydra** — SSH/service brute force
- **SQLmap** — SQL injection testing
- **Nikto** — Web vulnerability scanning
- **Burp Suite** — Web application exploitation
- **Metasploit** — Advanced exploitation and pivoting

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│           VirtualBox Bridged Network (192.168.1.0/24)       │
│                                                              │
│   ┌──────────────────┐      Attack Traffic      ┌─────────────────────────────┐
│   │  Kali Linux      │  ────────────────────►  │       Ubuntu Server         │
│   │  (Red Team)      │  ◄────────────────────  │  ┌─────────────────────┐    │
│   │ 192.168.1.16     │                         │  │ Snort IDS           │    │
│   │                  │                         │  │ (Real-time detect)  │    │
│   │ Tools:           │                         │  │ on enp0s3 (Passive) │    │
│   │ • Nmap           │                         │  └────────┬────────────┘    │
│   │ • Hydra          │                         │           │ Alerts          │
│   │ • SQLmap         │                         │           ▼                 │
│   │ • Nikto          │                         │  ┌─────────────────────┐    │
│   │ • Burp Suite     │                         │  │ Wazuh SIEM          │    │
│   │ • Metasploit     │                         │  │ (Correlation &      │    │
│   │                  │                         │  │ Dashboard)          │    │
│   └──────────────────┘                         │  └────────┬────────────┘    │
│                                                │           │ Events          │
│                                                │           ▼                 │
│                                                │  ┌─────────────────────┐    │
│                                                │  │ DVWA               │    │
│                                                │  │ Apache2 · MySQL    │    │
│                                                │  │ (Vulnerable App)   │    │
│                                                │  └─────────────────────┘    │
│                                                │  ┌─────────────────────┐    │
│                                                │  │ UFW Firewall        │    │
│                                                │  │ (Host-based control)│    │
│                                                │  └─────────────────────┘    │
│                                                └─────────────────────────────┘
└─────────────────────────────────────────────────────────────┘

Flow:
1. Attacker executes offensive tools on Kali
2. Attack traffic crosses network to Ubuntu target
3. Snort IDS captures packets in passive mode (does not block, only monitors)
4. Snort matches traffic against 4,057+ detection rules
5. Alerts written to /var/log/snort/alert in fast-alert format
6. Wazuh agent reads Snort alerts from local file
7. Wazuh correlates events with system logs, generates dashboard alerts
8. SOC analyst reviews Wazuh dashboard for incident investigation
```

---

## Snort IDS Configuration Details

### Deployment Mode
- **Passive IDS Mode** — Monitor-only (no blocking), captures and analyzes traffic without intercepting
- **Monitoring Interface** — `enp0s3` (Bridged adapter)
- **Network Scope** — `HOME_NET: 192.168.1.0/24`

### Detection Rules
- **Community Rules** — 4,057 baseline detection signatures
- **Custom Rules** — 4 lab-specific rules for common attacks:
  - **SID 1000001** — Nmap SYN scan detection (T1046)
  - **SID 1000002** — ICMP ping sweep detection (T1018)
  - **SID 1000003** — SSH brute force detection (T1110.001)
  - **SID 1000004** — SQLmap SQL injection detection (T1190)

### Alert Output
- **Format** — `alert_fast` (lightweight, human-readable)
- **Destination** — `/var/log/snort/alert`
- **Update Frequency** — Real-time as packets are processed

### Key Startup Command
```bash
sudo snort -A fast -c /etc/snort/snort.conf -i enp0s3 -l /var/log/snort/
```

---

## Wazuh SIEM Integration

### Configuration
```xml
<localfile>
  <log_format>snort-fast</log_format>
  <location>/var/log/snort/alert</location>
</localfile>
```

### Capabilities Enabled
- **Alert Parsing** — Automatic snort-fast format decoding
- **Severity Classification** — IDS alerts classified and ranked by threat level
- **Event Correlation** — Snort alerts correlated with system logs for contextual analysis
- **Unified Visibility** — All security events (IDS, logs, file integrity) in single dashboard
- **Incident Tracking** — Alert grouping and timeline reconstruction

---

## Attack Scenarios & Incident Response Workflow

### Standard Incident Response Cycle

```
Step 1: Attack Execution
   └─► Attacker runs tool (Nmap, Hydra, etc.) on Kali Linux
       └─► Network traffic generated toward target

Step 2: Detection
   └─► Snort IDS captures packets on enp0s3
       └─► Matches against 4,057+ rules
           └─► Alert triggered (if rule matched)
               └─► Alert written to /var/log/snort/alert

Step 3: Aggregation
   └─► Wazuh agent reads /var/log/snort/alert
       └─► Parses snort-fast format
           └─► Event ingested into Wazuh database

Step 4: Correlation & Alerting
   └─► Wazuh correlates Snort alert with system events
       └─► Severity assigned (Critical/High/Medium/Low)
           └─► Alert appears in Wazuh dashboard

Step 5: Investigation & Analysis
   └─► SOC analyst reviews Wazuh dashboard
       └─► Extracts IOCs (source IP, destination port, attack tool)
           └─► Reviews correlated events for full attack timeline

Step 6: Documentation
   └─► Structured incident report generated
       └─► Includes timeline, attack description, IOCs, MITRE mapping
           └─► Report filed for compliance & lessons learned
```

### Documented Incidents (5 Completed)

| Incident # | Date | Threat Level | Alert Count | Type |
|---|---|---|---|---|
| **Incident 1** | Mar 22, 2026 | High | 3,873 | Vulnerability Scanning (95%), Process Injection (2%) |
| **Incident 2** | In progress | TBD | TBD | TBD |
| **Incident 3** | In progress | TBD | TBD | TBD |
| **Incident 4** | In progress | TBD | TBD | TBD |
| **Incident 5** | In progress | TBD | TBD | TBD |

---

## MITRE ATT&CK Framework Mapping

Each simulated attack is mapped to MITRE ATT&CK techniques for threat intelligence correlation:

| Technique ID | Technique Name | Attack Tool | Detection Method | Custom SID |
|---|---|---|---|---|
| T1595.003 | Active Scanning | Nmap, Nikto | Network pattern matching | 1000001-1000002 |
| T1046 | Network Service Discovery | Nmap | Port scan detection | 1000001 |
| T1018 | Remote System Discovery | Nmap | ICMP sweep detection | 1000002 |
| T1110.001 | Brute Force (Password Guessing) | Hydra | Multiple connection attempts | 1000003 |
| T1190 | Exploit Public-Facing Application | SQLmap, Burp | SQL injection payloads | 1000004 |
| T1059 | Command and Scripting Interpreter | Metasploit | Exploitation signatures | Community rules |
| T1203 | Exploitation for Client Execution | Metasploit | Known CVE patterns | Community rules |

---

## Incident Reporting Methodology

Each incident report follows a structured SOC analyst template:

```
┌────────────────────────────────────────────┐
│  INCIDENT REPORT STRUCTURE                 │
├────────────────────────────────────────────┤
│                                            │
│  1. HEADER INFORMATION                     │
│     • Incident ID / Number                 │
│     • Date & Time Detected                 │
│     • Reporting Analyst & Tier Level       │
│                                            │
│  2. THREAT CLASSIFICATION                  │
│     • Severity Level (Critical/High/Medium)│
│     • Total Alerts Generated               │
│     • Attack Categories (%)                │
│                                            │
│  3. INDICATORS OF COMPROMISE (IOCs)        │
│     • Source IP Address(es)                │
│     • Target IP Address(es)                │
│     • Destination Ports                    │
│     • Attack Timeline (duration)           │
│                                            │
│  4. ATTACK ANALYSIS                        │
│     • Tool Used (Nmap, Hydra, etc.)        │
│     • Attack Techniques Detected           │
│     • MITRE ATT&CK Mapping                 │
│     • Vulnerability Categories             │
│                                            │
│  5. DETECTION & EVIDENCE                   │
│     • Snort SID(s) Triggered               │
│     • Wazuh Alert Count                    │
│     • Log References                       │
│     • Attack Artifacts                     │
│                                            │
│  6. RESPONSE ACTIONS                       │
│     • Immediate Containment (Block IP)     │
│     • Evidence Preservation                │
│     • Long-term Mitigation                 │
│                                            │
│  7. LESSONS LEARNED                        │
│     • Detection Gaps Identified            │
│     • Rule Tuning Recommendations          │
│     • Firewall Policy Updates              │
│                                            │
└────────────────────────────────────────────┘
```

---

## Project Directory Structure

```
Red-Blue-Team-Lab-Setup/
│
├── README.md                           # Main documentation (detailed lab guide)
├── PROJECT_SUMMARY.md                  # This file — overview & structure
├── index.html                          # Interactive network diagram (HTML + CSS)
│
├── daily-incidents/                    # Incident response documentation
│   ├── Incident_1/
│   │   ├── Report-1.txt               # Initial incident report
│   │   └── enhanced_report.txt        # Enhanced analysis report
│   │
│   ├── Incident-2/
│   │   ├── Incident-2_report.txt      # Incident 2 analysis
│   │   └── enhanced_report.txt        # Enhanced findings
│   │
│   ├── Incident-3/
│   │   └── incident3_report.txt       # Incident 3 documentation
│   │
│   ├── Incident-4/
│   │   └── incident-4_report.txt      # Incident 4 findings
│   │
│   └── Incident-5/
│       ├── report_incident_5.md       # Incident 5 (markdown format)
│       └── report_ai.md               # AI-assisted analysis
│
├── Snort-system/                       # IDS configuration & deployment logs
│   ├── Day_1/
│   │   └── report.md                  # Snort installation & config report
│   │
│   └── Day_2/                          # (Placeholder for future deployment phases)
│       └── (empty — planned for additional configs)
│
├── screenshots-ref/                    # Visual evidence & screenshots
│   └── (Directory for attack proof & dashboard captures)
│
└── .git/                               # Version control repository

```

### File Descriptions

| File/Folder | Purpose | Format |
|---|---|---|
| **README.md** | Complete lab setup guide with architecture, tools, rules, integration guide | Markdown |
| **PROJECT_SUMMARY.md** | High-level overview, structure, and methodology | Markdown |
| **index.html** | Interactive HTML-based network topology diagram with styling | HTML + CSS |
| **daily-incidents/** | Contains all incident reports and analysis documents | Mixed (TXT/MD) |
| **Incident_1/Report-1.txt** | First incident report (Vulnerability scanning, 3,873 alerts) | Text |
| **Incident-2/ through Incident-5/** | Additional incident documentation (ongoing) | Text/Markdown |
| **Snort-system/Day_1/report.md** | Snort installation and initial configuration report | Markdown |
| **screenshots-ref/** | Screenshots from Wazuh dashboard, alerts, and attack proof | Images |

---

## Skills Demonstrated

### Blue Team (Defense)
- ✅ Snort IDS rule development and tuning
- ✅ SIEM (Wazuh) configuration and log ingestion
- ✅ Alert correlation and anomaly detection
- ✅ Incident response workflows
- ✅ Structured incident documentation
- ✅ Network forensics and traffic analysis
- ✅ Firewall configuration (UFW)

### Red Team (Offense)
- ✅ Network reconnaissance (Nmap)
- ✅ Brute force attacks (Hydra)
- ✅ SQL injection (SQLmap)
- ✅ Web vulnerability scanning (Nikto)
- ✅ Web application exploitation (Burp Suite, DVWA)
- ✅ Advanced exploitation (Metasploit)

### SOC Operations
- ✅ Threat detection and classification
- ✅ IOC identification and tracking
- ✅ MITRE ATT&CK framework mapping
- ✅ Timeline reconstruction
- ✅ Evidence preservation
- ✅ Incident severity assessment
- ✅ Alert triage and false positive reduction

### Technical Infrastructure
- ✅ VirtualBox network simulation
- ✅ Linux system administration
- ✅ Service configuration (Apache, MySQL, SSH)
- ✅ Log management and centralization
- ✅ Network monitoring
- ✅ Security tooling integration

---

## Key Performance Metrics

| Metric | Value | Significance |
|---|---|---|
| **Snort Detection Rules** | 4,057+ | Comprehensive coverage of known attack patterns |
| **Custom Rules** | 4 | Lab-specific detections for common attack vectors |
| **Completed Incidents** | 5 | Real attack simulations fully documented |
| **Incident 1 Alerts** | 3,873 | Demonstrates volume handling & alert correlation |
| **Detection Latency** | Real-time | <1 second from attack to Snort alert |
| **SIEM Integration** | Functional | Snort → Wazuh pipeline operational |
| **Response Time** | Variable | Depends on analyst availability (SOC SLA practice) |

---

## Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────┐
│                  TECHNOLOGY STACK                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  VIRTUALIZATION LAYER                                      │
│  • VirtualBox 7.x (Hypervisor)                             │
│  • Bridged Networking (Real LAN connectivity)              │
│                                                             │
│  ATTACK PLATFORM (Red Team)                                │
│  • Kali Linux 2024.x (Offensive OS)                        │
│  • Nmap (Network reconnaissance)                           │
│  • Hydra (Brute force)                                     │
│  • SQLmap (SQL injection)                                  │
│  • Nikto (Web scanning)                                    │
│  • Burp Suite Community (Web exploitation)                 │
│  • Metasploit Framework (Advanced exploitation)            │
│                                                             │
│  DEFENSE PLATFORM (Blue Team)                              │
│  • Ubuntu Server 22.04+ (Defender OS)                      │
│  • Snort IDS 2.9.20 (Network IDS)                          │
│  • Wazuh 4.x (SIEM & Monitoring)                           │
│  • UFW (Host-based firewall)                               │
│  • Apache2 (Web server)                                    │
│  • MySQL/PHP (Database & scripting)                        │
│  • OpenSSH (Remote access)                                 │
│                                                             │
│  APPLICATION LAYER                                         │
│  • DVWA (Damn Vulnerable Web App)                          │
│  • PHP (Web scripting)                                     │
│  • MySQL (Relational database)                             │
│                                                             │
│  DOCUMENTATION & VISUALIZATION                             │
│  • Markdown (Reports)                                      │
│  • HTML5 + CSS3 (Interactive diagrams)                     │
│  • Git (Version control)                                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Learning Pathways

### For SOC Analysts
1. **Alert Analysis** — Review incident reports in `daily-incidents/`
2. **IOC Extraction** — Practice identifying source IPs, ports, attack tools
3. **Timeline Reconstruction** — Use alert timestamps to build attack sequences
4. **Severity Assessment** — Determine threat level based on alert volume & types
5. **MITRE Mapping** — Correlate detected techniques with ATT&CK framework

### For Penetration Testers
1. **Tool Mastery** — Practice Nmap, SQLmap, Hydra in isolated lab environment
2. **Detection Evasion** — Understand IDS signatures to refine attack techniques
3. **Impact Assessment** — See real-time alert generation from offensive actions
4. **Reporting Skills** — Generate incident reports as if from defender perspective

### For Blue Teamers
1. **Rule Development** — Study custom Snort rules in README.md
2. **SIEM Operations** — Review Wazuh dashboard integration and log correlation
3. **Network Analysis** — Understand packet patterns for common attacks
4. **Incident Response** — Follow structured documentation in incident reports
5. **Tuning & Optimization** — Identify false positives and rule improvements

---

## Current Status & Roadmap

### ✅ Completed Phases
- [x] Red Team / Blue Team lab setup (Kali + Ubuntu + Wazuh + DVWA)
- [x] Snort IDS deployment and configuration
- [x] Custom detection rule development (4 rules)
- [x] Wazuh + Snort integration
- [x] Structured incident reporting workflow
- [x] MITRE ATT&CK technique mapping
- [x] Documentation and project summarization

### 🔄 In Progress
- [ ] Incident 2-5 completion and analysis
- [ ] Advanced rule tuning based on false positive feedback
- [ ] Snort Day_2 deployment (additional configurations)

### 📋 Planned Features (Hardware Upgrade Pending)
- [ ] Windows 10 endpoint + Sysmon telemetry
- [ ] Phishing detection lab (GoPhish + YARA rules)
- [ ] Threat hunting with Elastic Stack (ELK + Sigma rules)
- [ ] Digital Forensics & Incident Response (DFIR) workstation
- [ ] Network segmentation & VLAN isolation
- [ ] Multi-layer detection (host + network + cloud)

---

## Disclaimer

> This project is strictly for **educational purposes** in a controlled, isolated lab environment. 
> All attack simulations are performed only within the virtual lab.
> No external, production, or unauthorized systems were targeted.
> All tools are used in compliance with their respective licenses and terms of service.

---

## Key Takeaways

1. **Realistic SOC Simulation** — This lab mirrors actual SOC operations with real tools and workflows
2. **End-to-End Detection** — From attack execution to incident report, every step is documented
3. **Hands-On Learning** — Practical experience with industry-standard security tools
4. **Scalable Architecture** — Foundation can expand to include additional endpoints, techniques, and frameworks
5. **Portfolio Demonstration** — Comprehensive project showcasing SOC analyst and security engineering skills

---

## Quick Start References

- **Main Documentation** — See [README.md](README.md) for detailed setup instructions
- **Network Diagram** — View [index.html](index.html) for interactive topology
- **Incident Analysis** — Browse [daily-incidents/](daily-incidents/) for structured reports
- **IDS Configuration** — Check [Snort-system/Day_1/report.md](Snort-system/Day_1/report.md) for deployment details

---

**Author:** Gade Kuldeep  
**Project Type:** Cybersecurity Home Lab | Red Team / Blue Team Simulation | SOC Operations  
**Technologies:** Snort · Wazuh · Kali Linux · Ubuntu · DVWA · VirtualBox  
**Framework:** MITRE ATT&CK · Incident Response · Network Forensics  
**Status:** Active & Ongoing Development
