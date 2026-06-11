# 🛡 Incident Ticket: SOC-2026-INC6-001

**Severity:** Medium
**Status:** Closed
**Created By:** SOC L1 Analyst
**Assigned To:** Detection Engineering / SOC Team
**Date/Time:** [Lab Execution Date]

---

## 📌 Summary

SSH-related activity was detected against the Ubuntu Server within the SOC lab environment. The activity was intentionally generated from a Kali Linux attack machine to simulate attack traffic, validate Snort IDS detections, and develop custom monitoring rules.

The exercise included firewall configuration, SSH service exposure, Snort alert monitoring, custom rule creation, and Wazuh SIEM log analysis.

---

## 🌐 Affected Assets

### Target System

* Host: Ubuntu Server
* IP Address: 192.168.1.16
* Service: OpenSSH
* Port: 22/TCP

### Source System

* Host: Kali Linux
* Role: Attack Simulation Machine

---

## 🔍 Detection Details

### Detection Sources

* Snort IDS
* Wazuh SIEM
* Ubuntu System Logs
* UFW Firewall Logs

### Detection Method

* Existing Snort TCP Scan Detection Rule
* Custom SSH Detection Rule
* Wazuh Alert Correlation

### Generated Alerts

```text
TCP Scan Detected
```

```text
SSH Port Under Attack
```

---

## 🧠 Analysis

During the investigation, SSH traffic was generated from the Kali Linux system toward the Ubuntu Server.

Observed activity included:

* Traffic targeting TCP Port 22
* Successful SSH connection attempts
* Snort IDS alert generation
* Wazuh event collection and correlation

Initial analysis identified traffic patterns associated with SSH communication and service interaction.

Existing Snort rules detected suspicious TCP activity.

Further investigation showed repeated traffic directed toward the SSH service, leading to the creation of a dedicated SSH monitoring rule.

No indicators of unauthorized access or system compromise were identified.

---

## 📂 Evidence

### Firewall Configuration

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
```

### SSH Service Status

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

### Existing Snort Rule

```snort
alert tcp any any -> 192.168.1.16 any (
msg:"TCP Scan Detected";
flags:S;
threshold:type threshold, track by_src, count 20, seconds 3;
sid:10000003;
rev:1;
)
```

### Custom SSH Detection Rule

```snort
alert tcp any any -> 192.168.1.16 22 (
msg:"SSH Port Under Attack";
sid:10000008;
rev:1;
)
```

### Log Sources Reviewed

```text
/var/log/auth.log
```

```text
/var/log/syslog
```

```text
/var/log/ufw.log
```

```text
/var/log/snort/
```

### Wazuh Events

Observed alerts successfully forwarded and visible within Wazuh SIEM.

---

## 🛡 Actions Taken

* Configured UFW firewall policies
* Allowed SSH service through firewall
* Enabled OpenSSH service
* Generated SSH traffic from Kali Linux
* Monitored Snort IDS alerts
* Investigated generated logs
* Reviewed existing detection rules
* Developed custom SSH detection rule
* Validated alert generation
* Correlated alerts through Wazuh SIEM

---

## ⚠️ Impact Assessment

### Impact Level

Low

### Findings

* SSH service intentionally exposed for testing
* Activity originated from authorized internal lab machine
* No privilege escalation observed
* No unauthorized access detected
* No system compromise identified

The activity was part of a controlled cybersecurity laboratory exercise.

---

## ✅ Conclusion

Investigation confirmed that the observed SSH activity originated from an authorized attack simulation conducted within the SOC lab environment.

Snort IDS successfully detected network activity and generated alerts. A dedicated SSH monitoring rule was created and validated. Alerts were successfully collected and monitored through Wazuh SIEM.

No malicious activity or compromise was observed.

---

## 📌 Recommendations

* Continue development of custom Snort detection rules
* Improve SSH monitoring coverage
* Maintain centralized log collection through Wazuh
* Regularly review firewall policies
* Document detection engineering workflows
* Expand lab scenarios to include additional attack techniques

---

## 🔄 Next Steps

* Mark activity as Authorized Security Testing
* Archive logs and evidence
* Update SOC detection documentation
* Continue development of advanced detection rules
* Proceed with next Red Team vs Blue Team incident simulation

---

## Incident Status

✅ Closed

Classification: Authorized Security Testing / SOC Lab Activity

No compromise detected.
