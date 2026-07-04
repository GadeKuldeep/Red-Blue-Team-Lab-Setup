# Phishing Simulation — GoPhish + Wazuh Integration

This folder documents the phishing simulation component of the CyberOps SOC Home Lab
(Red-Blue-Team-Lab-Setup). It covers standing up a GoPhish campaign, routing its logs
into the existing Wazuh SIEM, and writing custom detection rules so simulated phishing
activity is visible in the same pipeline as the rest of the lab's alerts.

## Objective

Most of this lab focuses on the network/host detection side (Snort + Wazuh). This
folder adds the human-risk layer: simulating a phishing campaign end-to-end and proving
that campaign activity (emails sent, opened, links clicked, credentials submitted) can be
captured, logged, and alerted on — not just observed from the GoPhish dashboard.

## Tech Stack

| Component | Role |
|---|---|
| **GoPhish** | Open-source phishing simulation framework — builds and tracks the campaign |
| **Wazuh Manager** | SIEM — ingests GoPhish logs and raises alerts via custom rules |
| **Ubuntu Server** | Host running GoPhish and the Wazuh agent/manager |
| **SQLite3** | Default GoPhish backend database (`gophish.db`) |

## Setup Summary

### 1. GoPhish deployment
GoPhish was deployed at `/opt/gophish` and run directly from source (`./gophish`).

Key config (`config.json`):
- Admin server: `https://127.0.0.1:3333` (TLS, self-signed cert)
- Phishing server: `http://0.0.0.0:80` (serves the actual landing pages to targets)
- Backend: SQLite3 (`gophish.db`)
- Logging: `/var/log/gophish.log` at `info` level

### 2. Campaign components built
- **Email Template** — the pretext email sent to targets
- **Landing Page** — the fake credential-capture page targets are redirected to
- **Sending Profile** — SMTP configuration used to actually deliver the emails
- **Target Group** — the list of simulated recipients for the campaign

### 3. Campaign execution
Ran a campaign against a small internal target group and tracked it through GoPhish's
built-in metrics:

| Metric | Result |
|---|---|
| Emails Sent | 5 |
| Emails Opened | 1 |
| Links Clicked | 1 |
| Data Submitted | 0 |
| Reported as Phishing | 0 |

This gives a simple, realistic click-through funnel to reference when discussing
phishing susceptibility rates.

### 4. Wazuh integration
GoPhish doesn't have native SIEM output, so its log file was wired into Wazuh manually
via `ossec.conf`, alongside the existing Apache and Snort log sources already configured
in the lab:

```xml
<localfile>
  <location>/var/log/gophish.log</location>
  <log_format>syslog</log_format>
</localfile>
```

### 5. Custom Wazuh detection rules
Custom rules were added in `local_rules.xml` to decode and alert on GoPhish log events,
following the same severity-tiering approach used for the Snort rules elsewhere in this
lab:

| Rule ID | Level | Trigger | Purpose |
|---|---|---|---|
| 100100 | 0 | Any GoPhish log line | Base rule — groups all GoPhish events under one decoder |
| 100101 | 3 | `log_level=warning` | Low-severity operational warnings (e.g. missing contact address config) |
| 100102 | 7 | `log_level=error` | Errors during campaign operation |
| 100103 | 12 | `log_level=fatal` | Fatal crashes — highest severity, immediate attention |

This mirrors real-world MDR/SOC practice: a phishing simulation platform's own operational
logs are themselves a log source worth monitoring, separate from the campaign metrics it
reports on its own dashboard.

## What This Demonstrates

- Standing up and operating an open-source phishing simulation tool end-to-end
- Bridging a non-native log source into an existing SIEM pipeline
- Writing tiered custom detection rules (info → warning → error → fatal)
- Thinking about phishing simulation from both sides: as the "attacker" building the
  campaign, and as the SOC analyst who would need visibility into it operationally

## Notes / Next Steps

- Add a rule that specifically fires on a `clicked_link` or `submitted_data` event
  parsed from GoPhish's structured logs, rather than only its operational log levels
- Extend the sending profile to route through a dedicated test mailbox rather than a
  personal address
- Correlate GoPhish "link clicked" timestamps against Wazuh/Snort network alerts to see
  if the simulated click generates any corresponding network-layer signal