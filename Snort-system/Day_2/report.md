# Snort IDS Integration with Wazuh SIEM

## Overview
This project demonstrates the integration of Snort IDS with Wazuh SIEM in a small SOC lab environment.  
The setup was used to generate, detect, and monitor network-based attacks using custom Snort rules.

---

# Lab Setup

## Machines Used

| Machine | Purpose |
|--------|---------|
| Kali Linux VM | Attack machine |
| Ubuntu Server VM | Snort IDS + Wazuh SIEM |

---

# Technologies Used

- Kali Linux
- Ubuntu Server
- Snort IDS
- Wazuh SIEM
- VirtualBox Bridged Network

---

# Network Configuration

- Both virtual machines were connected using a bridged network.
- Kali Linux and Ubuntu Server were configured on different IP addresses for attack simulation and monitoring.

---

# Snort IDS Configuration

## Installed Snort IDS on Ubuntu Server

Snort was configured to:
- Monitor network traffic
- Detect suspicious ICMP traffic
- Generate alerts based on custom rules

---

# Custom Rule Development

## ICMP Rule Testing

Worked with:
```bash
icmp.rules
```

Created custom ICMP detection rules to:
- Detect ping traffic
- Generate alerts when ICMP packets were received

Example alert action used:
```bash
alert icmp any any -> any any
```

---

# Attack Simulation

## Traffic Generation from Kali Linux

Generated ICMP traffic from Kali Linux to Ubuntu Server using ping commands.

Purpose:
- Trigger Snort alerts
- Verify IDS detection capability

---

# Wazuh Integration

Integrated Snort alerts with Wazuh SIEM for:
- Centralized monitoring
- Alert visibility
- Log analysis

Verified that generated Snort alerts were successfully visible inside Wazuh.

---

# Outcome

Successfully:
- Configured Snort IDS
- Created custom ICMP rules
- Generated attack traffic
- Triggered alerts
- Monitored alerts in Wazuh SIEM
- Documented detection workflow for SOC portfolio

---

# Future Improvements

- Add more custom Snort rules
- Detect brute-force attempts
- Detect port scanning activity
- Integrate Sysmon logs
- Create incident response reports
