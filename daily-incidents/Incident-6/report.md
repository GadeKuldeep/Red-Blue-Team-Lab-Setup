# Incident-6: SSH Attack Detection, Log Analysis and SIEM Monitoring

## Overview

This incident was conducted in a controlled SOC lab environment to simulate SSH-based attack activity, analyze generated logs, monitor alerts through Snort IDS and Wazuh SIEM, and improve detection engineering skills through custom rule development.

The objective was to understand the complete attack-to-detection lifecycle, including firewall configuration, service exposure, network monitoring, alert generation, and centralized log analysis.

---

# Incident Information

| Field           | Value                 |
| --------------- | --------------------- |
| Incident ID     | Incident-6            |
| Incident Type   | SSH Attack Simulation |
| Severity        | Medium                |
| Status          | Closed                |
| Detection Tools | Snort IDS, Wazuh SIEM |
| Attack Machine  | Kali Linux            |
| Target Machine  | Ubuntu Server         |

---

# Lab Environment

## Machines Used

| Machine          | Purpose                 |
| ---------------- | ----------------------- |
| Kali Linux VM    | Attack Machine          |
| Ubuntu Server VM | Snort IDS + Wazuh Agent |

---

# Technologies Used

* Kali Linux
* Ubuntu Server
* Snort IDS
* Wazuh SIEM
* OpenSSH Server
* UFW Firewall
* VirtualBox

---

# Network Configuration

Both virtual machines were connected through the same VirtualBox network.

## Target System

Ubuntu Server

Example IP Address:

```text
192.168.1.16
```

## Attacker System

Kali Linux

Used for generating SSH traffic toward the target system.

---

# Environment Preparation

## Firewall Configuration

Configured default firewall policies using UFW.

Block incoming traffic:

```bash
sudo ufw default deny incoming
```

Allow outgoing traffic:

```bash
sudo ufw default allow outgoing
```

Verification:

```bash
sudo ufw status verbose
```

Purpose:

* Restrict unnecessary inbound traffic
* Allow controlled access to services

---

## SSH Port Configuration

Allowed SSH service through firewall.

```bash
sudo ufw allow 22/tcp
```

Verification:

```bash
sudo ufw status numbered
```

Purpose:

* Permit SSH communication
* Create attack simulation environment

---

# SSH Service Configuration

Enabled SSH service on Ubuntu Server.

```bash
sudo systemctl enable --now ssh
```

Verification:

```bash
sudo systemctl status ssh
```

Purpose:

* Accept SSH connections
* Generate logs for analysis

---

# Snort IDS Configuration

Snort IDS was configured to monitor network traffic on the Ubuntu Server.

Configuration validation:

```bash
sudo snort -T -c /etc/snort/snort.conf
```

Purpose:

* Inspect network traffic
* Generate alerts for suspicious activity
* Support attack monitoring

---

# Existing Snort Rule Analysis

Reviewed an existing TCP scan detection rule.

Rule:

```snort
alert tcp any any -> 192.168.1.16 any (
msg:"TCP Scan Detected";
flags:S;
threshold:type threshold, track by_src, count 20, seconds 3;
sid:10000003;
rev:1;
)
```

Purpose:

* Detect rapid TCP SYN activity
* Identify possible scanning behavior

Generated Alert:

```text
TCP Scan Detected
```

---

# SSH Attack Simulation

Generated SSH traffic from Kali Linux.

Example Command:

```bash
ssh user@192.168.1.16
```

Purpose:

* Create SSH-related network events
* Observe IDS detection
* Analyze generated logs

Observed Activity:

* SSH connection attempts
* TCP communication on Port 22
* Network packets captured by Snort

---

# Custom Snort Rule Development

After reviewing generated traffic, created a dedicated SSH detection rule.

Rule:

```snort
alert tcp any any -> 192.168.1.16 22 (
msg:"SSH Port Under Attack";
sid:10000008;
rev:1;
)
```

Purpose:

* Detect traffic directed toward SSH service
* Generate dedicated SSH alerts
* Improve monitoring visibility

---

# Rule Validation

Restarted Snort monitoring and generated SSH traffic again.

Observed Alert:

```text
SSH Port Under Attack
```

Result:

The custom rule successfully generated alerts when SSH traffic was detected.

---

# Snort Log Analysis

Monitored generated alerts through Snort.

Observed Information:

* Source IP Address
* Destination IP Address
* Protocol
* Destination Port
* Alert Message
* Timestamp

Analysis Focus:

* SSH traffic patterns
* Detection accuracy
* Alert generation behavior

---

# Wazuh Integration

Snort alerts were forwarded to Wazuh SIEM for centralized monitoring.

Purpose:

* Collect security alerts
* Centralize log management
* Improve visibility of network events

Verified:

* Snort alerts appeared in Wazuh dashboard
* Alert information was searchable
* Event timestamps were recorded correctly

---

# Wazuh Log Analysis

Analyzed generated events inside Wazuh.

Observed Fields:

* Rule ID
* Alert Level
* Source Information
* Event Time
* Snort Alert Message

Examples:

```text
TCP Scan Detected
```

```text
SSH Port Under Attack
```

Purpose:

* Correlate IDS events
* Improve SOC investigation workflow
* Validate detection pipeline

---

# Investigation Summary

The generated SSH traffic was successfully detected by Snort IDS.

Log analysis confirmed:

* SSH service exposure on Port 22
* Detection of SSH-related traffic
* Successful generation of custom alerts
* Proper forwarding of alerts to Wazuh

No unauthorized access occurred.

The activity was intentionally generated for SOC training and detection engineering purposes.

---

# Actions Performed

* Configured UFW firewall
* Allowed SSH traffic
* Enabled SSH service
* Generated SSH traffic from Kali Linux
* Reviewed Snort alerts
* Analyzed network logs
* Examined existing TCP scan rule
* Developed custom SSH detection rule
* Validated alert generation
* Monitored events inside Wazuh
* Investigated generated logs

---

# Outcome

Successfully:

✅ Configured firewall policies

✅ Enabled SSH service

✅ Generated attack traffic

✅ Monitored network events

✅ Analyzed Snort alerts

✅ Created custom detection rule

✅ Validated detection logic

✅ Observed alerts in Wazuh

✅ Improved SOC investigation workflow

✅ Gained practical detection engineering experience

---

# Lessons Learned

* SSH services are common monitoring targets.
* Snort rules can be customized to improve visibility.
* Firewall configuration directly affects attack surface exposure.
* Wazuh provides centralized alert visibility.
* Log analysis is essential for SOC operations.
* Detection engineering improves monitoring effectiveness.

---

# Incident Status

Closed

The incident was successfully detected, analyzed, and documented within the SOC lab environment.

No system compromise occurred.

The activity was performed for cybersecurity training, alert analysis, and detection engineering practice.
