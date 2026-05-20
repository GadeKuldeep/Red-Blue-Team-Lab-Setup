# 🛡️ Day 01 — Snort IDS Installation & Configuration

## Overview

This document contains the Day 01 implementation report for the Red Team / Blue Team SOC Home Lab project.

The objective of this phase was to successfully install and configure Snort IDS on the Ubuntu Server environment and prepare the system for future attack detection and monitoring activities.

This report only covers:
- Snort installation
- Initial configuration
- Logging setup
- Validation testing
- IDS mode deployment

Custom Snort rule creation and attack detection activities are documented separately in the Day 02 report.

---

# Lab Environment

| Component | Details |
|---|---|
| Attacker Machine | Kali Linux |
| Defender Machine | Ubuntu Server |
| IDS Tool | Snort 2.9 |
| Virtualization | VirtualBox |
| Network Type | Bridged Adapter |
| Monitoring Interface | `enp0s3` |

---

# Objectives Completed

- [x] Ubuntu Server prepared
- [x] Snort IDS installed
- [x] Snort configuration completed
- [x] HOME_NET configured
- [x] Logging directory configured
- [x] Snort configuration validated
- [x] Snort successfully started in IDS mode

---

# System Update

Before installation, the Ubuntu Server packages were updated.

```bash
sudo apt update && sudo apt upgrade -y
```

---

# Snort Installation

Snort IDS was installed using the Ubuntu package manager.

```bash
sudo apt install snort -y
```

During installation:
- monitoring interface was selected,
- HOME_NET was configured,
- default Snort rules were installed.

---

# Main Configuration File

```bash
/etc/snort/snort.conf
```

---

# HOME_NET Configuration

The protected internal network range was configured.

```bash
ipvar HOME_NET 192.168.1.1/24
```

---

# Rule Path Configuration

Snort rule directory verification:

```bash
var RULE_PATH /etc/snort/rules
```

---

# Alert Logging Configuration

Fast alert logging was enabled.

```bash
output alert_fast: snort.alert.fast
```

---

# Log Directory

Snort logs were stored in:

```bash
/var/log/snort/
```

---

# Configuration Validation

Snort configuration testing was performed to verify:
- syntax correctness,
- interface configuration,
- rule loading,
- detection engine initialization.

## Validation Command

```bash
sudo snort -T -c /etc/snort/snort.conf -i enp0s3
```

## Validation Result

The configuration validation completed successfully without critical errors.

---

# Running Snort in IDS Mode

Snort was started in passive IDS monitoring mode.

## IDS Startup Command

```bash
sudo snort -A fast -c /etc/snort/snort.conf -i enp0s3 -l /var/log/snort/
```

## Result

Snort successfully:
- initialized packet capture,
- loaded rules,
- started monitoring live traffic,
- generated logging output.

---

# Key Observations

- Snort IDS deployment completed successfully.
- Logging functionality was operational.
- The bridged network setup enabled visibility between Kali Linux and Ubuntu Server.
- The environment was successfully prepared for future attack simulations and detection testing.

---

# Challenges Faced

| Issue | Resolution |
|---|---|
| Alert file not generated initially | Restarted Snort and verified logging configuration |
| Network interface verification required | Confirmed interface using `ip addr` |
| Log permissions verification | Checked `/var/log/snort/` access |

---

# Conclusion

Day 01 focused on establishing the IDS foundation for the SOC Home Lab environment.

The installation and configuration of Snort IDS were completed successfully, and the system was prepared for:
- custom rule development,
- attack simulation,
- alert monitoring,
- SIEM integration,
- future detection engineering activities.

The environment is now ready for Day 02 activities involving custom Snort rule creation and attack detection testing.