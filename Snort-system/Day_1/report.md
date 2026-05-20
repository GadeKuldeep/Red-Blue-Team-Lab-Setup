# 🛡️ Snort IDS — Installation & Configuration Report
**Platform:** Ubuntu 22.04 LTS  
**Snort Version:** 3.x (Snort3)  
**Date:** 2026-05-20  
**Author:** Security Engineering Team  
**Classification:** Internal / Lab Documentation

---

## Table of Contents

1. [Overview](#overview)
2. [Environment Details](#environment-details)
3. [Pre-Installation Steps](#pre-installation-steps)
4. [Installing Dependencies](#installing-dependencies)
5. [Installing Snort 3](#installing-snort-3)
6. [Configuration](#configuration)
7. [Writing & Testing Rules](#writing--testing-rules)
8. [Running Snort](#running-snort)
9. [Setting Snort as a Systemd Service](#setting-snort-as-a-systemd-service)
10. [Log Analysis](#log-analysis)
11. [Verification & Testing](#verification--testing)
12. [Troubleshooting](#troubleshooting)
13. [Security Hardening Notes](#security-hardening-notes)
14. [Summary & Findings](#summary--findings)

---

## Overview

Snort is an open-source **Network Intrusion Detection/Prevention System (NIDS/NIPS)** developed by Cisco. It performs real-time traffic analysis and packet logging on IP networks using a combination of protocol analysis, content searching, and various pre-processors to detect probes, attacks, and anomalies.

This report documents a **day-in-the-field** exercise covering:
- Full installation on Ubuntu 22.04 from source
- Configuration for passive IDS mode
- Custom rule writing
- Live traffic testing
- Systemd service deployment

---

## Environment Details

| Parameter        | Value                          |
|------------------|-------------------------------|
| OS               | Ubuntu 22.04.3 LTS (Jammy)    |
| Kernel           | 5.15.0-91-generic              |
| Snort Version    | 3.1.74.0                       |
| Interface        | `ens33` (adjust to yours)      |
| Log Directory    | `/var/log/snort/`              |
| Config Path      | `/etc/snort/snort.lua`         |
| Rules Path       | `/etc/snort/rules/`            |
| Network CIDR     | `192.168.1.0/24` (example)     |
| Mode             | Passive IDS (AFPACKET)         |

---

## Pre-Installation Steps

### 1. Update the System

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential git cmake autoconf libtool pkg-config
```

### 2. Verify Network Interface

```bash
ip link show
# Note the interface name — commonly ens33, eth0, enp0s3
ip addr show ens33
```

### 3. Set Interface to Promiscuous Mode

```bash
sudo ip link set ens33 promisc on

# Verify
ip link show ens33 | grep promisc
```

---

## Installing Dependencies

Snort 3 requires several libraries. Install them in the following order:

### 1. Core Build Tools & Libraries

```bash
sudo apt install -y \
  libpcap-dev libpcre2-dev libdnet-dev \
  zlib1g-dev liblzma-dev openssl libssl-dev \
  libhwloc-dev libnuma-dev \
  bison flex \
  libluajit-5.1-dev luajit \
  libsqlite3-dev uuid-dev
```

### 2. Install LibDAQ (Data Acquisition Library)

LibDAQ is Snort's packet I/O abstraction layer.

```bash
cd /tmp
git clone https://github.com/snort3/libdaq.git
cd libdaq
./bootstrap
./configure
make -j$(nproc)
sudo make install
sudo ldconfig
```

Verify DAQ installation:

```bash
daq-modules
# Expected output: afpacket, bpf, dump, fst, gwlb, nfq, pcap, savefile, trace
```

### 3. Install Hyperscan (High-Performance Regex Engine)

```bash
sudo apt install -y libhyperscan-dev
# OR build from source if package not available:
# cd /tmp && git clone https://github.com/intel/hyperscan.git
# cmake -DCMAKE_BUILD_TYPE=Release /tmp/hyperscan
# make -j$(nproc) && sudo make install
```

---

## Installing Snort 3

### 1. Download Source

```bash
cd /tmp
wget https://github.com/snort3/snort3/archive/refs/tags/3.1.74.0.tar.gz
tar -xzf 3.1.74.0.tar.gz
cd snort3-3.1.74.0
```

### 2. Configure & Compile

```bash
./configure_cmake.sh --prefix=/usr/local --enable-tcmalloc
cd build
make -j$(nproc)
sudo make install
sudo ldconfig
```

### 3. Verify Installation

```bash
snort -V
# Expected:
#    ,,_     -*> Snort++ <*-
#   o"  )~   Version 3.1.74.0
#   ''''      By Martin Roesch & The Snort Team
```

---

## Configuration

### 1. Create Directory Structure

```bash
sudo mkdir -p /etc/snort/rules
sudo mkdir -p /var/log/snort
sudo touch /etc/snort/rules/local.rules
sudo chown -R $USER:$USER /etc/snort
```

### 2. Copy Default Config

```bash
sudo cp /usr/local/etc/snort/snort.lua /etc/snort/snort.lua
sudo cp /usr/local/etc/snort/snort_defaults.lua /etc/snort/snort_defaults.lua
```

### 3. Edit `snort_defaults.lua` — Define Home Network

```bash
sudo nano /etc/snort/snort_defaults.lua
```

Update the HOME_NET variable:

```lua
-- HOME_NET: Your protected network CIDR
HOME_NET = '192.168.1.0/24'

-- EXTERNAL_NET: Everything that is not HOME_NET
EXTERNAL_NET = '!$HOME_NET'

-- Rule paths
RULE_PATH         = '/etc/snort/rules'
BUILTIN_RULE_PATH = '/usr/local/etc/snort/builtin_rules'
```

### 4. Edit `snort.lua` — Core Configuration

```bash
sudo nano /etc/snort/snort.lua
```

Key sections to configure:

```lua
-- ===== Network Interface =====
-- (passed as CLI argument, not in config)

-- ===== Alert Output =====
alert_fast =
{
    file = true,
    packet = false,
    limit = 10,
}

-- ===== Logging =====
-- Unified2 format for external tools (Barnyard2, SIEM)
-- Outputs to /var/log/snort/

-- ===== IPS Mode Toggle =====
-- For passive IDS, ensure ips.mode is NOT set to 'inline'
ips =
{
    enable_builtin_rules = true,
    include = RULE_PATH .. '/local.rules',
    variables = default_variables,
}

-- ===== Detection Engine =====
detection =
{
    hyperscan_literals = true,
    pcre_to_regex = false,
}
```

### 5. Validate Config

```bash
snort -c /etc/snort/snort.lua --warn-all
# Should output: Snort successfully validated the configuration
```

---

## Writing & Testing Rules

### Snort 3 Rule Syntax

```
action proto src_ip src_port direction dst_ip dst_port ( options; )
```

### 1. Create Local Rules File

```bash
sudo nano /etc/snort/rules/local.rules
```

### 2. Example Rules

```snort
# --------------------------------------------------
# ICMP Rules
# --------------------------------------------------

# Detect any ICMP ping to HOME_NET
alert icmp any any -> $HOME_NET any (
    msg:"ICMP Ping Detected";
    itype:8;
    sid:1000001;
    rev:1;
)

# Detect ICMP ping sweep (nmap -sn style)
alert icmp any any -> $HOME_NET any (
    msg:"ICMP Ping Sweep Detected";
    itype:8;
    threshold: type both, track by_src, count 10, seconds 5;
    sid:1000002;
    rev:1;
)

# --------------------------------------------------
# TCP Rules
# --------------------------------------------------

# Detect Nmap SYN scan (half-open scan)
alert tcp any any -> $HOME_NET any (
    msg:"Possible Nmap SYN Scan";
    flags:S;
    threshold: type both, track by_src, count 50, seconds 3;
    sid:1000003;
    rev:1;
)

# Detect SSH brute force
alert tcp any any -> $HOME_NET 22 (
    msg:"SSH Brute Force Attempt";
    flags:S;
    threshold: type both, track by_src, count 5, seconds 10;
    sid:1000004;
    rev:1;
)

# Detect Telnet connections (deprecated protocol)
alert tcp any any -> $HOME_NET 23 (
    msg:"Telnet Connection Attempt";
    flags:S;
    sid:1000005;
    rev:1;
)

# --------------------------------------------------
# HTTP Rules
# --------------------------------------------------

# Detect SQL Injection attempt patterns in HTTP
alert http any any -> $HOME_NET 80 (
    msg:"Possible SQL Injection - UNION SELECT";
    http_uri;
    content:"UNION SELECT", nocase;
    sid:1000006;
    rev:1;
)

# Detect XSS pattern in HTTP request
alert http any any -> $HOME_NET 80 (
    msg:"Possible XSS Attack - script tag";
    http_uri;
    content:"<script>", nocase;
    sid:1000007;
    rev:1;
)

# Detect directory traversal
alert http any any -> $HOME_NET 80 (
    msg:"Directory Traversal Attempt";
    http_uri;
    content:"../";
    sid:1000008;
    rev:1;
)

# --------------------------------------------------
# DNS Rules
# --------------------------------------------------

# Detect DNS amplification-style query (ANY records)
alert udp any any -> any 53 (
    msg:"DNS ANY Query - Possible Amplification";
    byte_test:1,&,0x78,2;
    sid:1000009;
    rev:1;
)

# --------------------------------------------------
# FTP Rules
# --------------------------------------------------

# Detect anonymous FTP login
alert tcp any any -> $HOME_NET 21 (
    msg:"Anonymous FTP Login Attempt";
    content:"USER anonymous";
    nocase;
    sid:1000010;
    rev:1;
)
```

### 3. Test Rules Syntax

```bash
snort -c /etc/snort/snort.lua -T
# -T flag = test/validate only, no packet capture
```

---

## Running Snort

### Mode 1: Packet Sniffer (Quick Test)

```bash
sudo snort -i ens33 -v
# Shows live packet headers
```

### Mode 2: IDS with Alerts (Console Output)

```bash
sudo snort -c /etc/snort/snort.lua -i ens33 -A alert_fast -s 65535 -k none
```

Flag breakdown:

| Flag             | Description                              |
|------------------|------------------------------------------|
| `-c`             | Path to config file                      |
| `-i ens33`       | Capture interface                        |
| `-A alert_fast`  | Alert output mode (fast = one-line)      |
| `-s 65535`       | Snaplen (capture entire packets)         |
| `-k none`        | Disable checksum validation (VMs)        |

### Mode 3: IDS with File Logging

```bash
sudo snort -c /etc/snort/snort.lua \
    -i ens33 \
    -l /var/log/snort \
    -A alert_fast \
    -s 65535 \
    -k none \
    --daq afpacket \
    --daq-var buffer_size=256
```

### Mode 4: Read from PCAP (Offline Analysis)

```bash
sudo snort -c /etc/snort/snort.lua \
    -r /path/to/capture.pcap \
    -A alert_fast \
    -l /var/log/snort
```

---

## Setting Snort as a Systemd Service

### 1. Create Service File

```bash
sudo nano /etc/systemd/system/snort3.service
```

```ini
[Unit]
Description=Snort3 NIDS
After=network.target
Wants=network-online.target

[Service]
Type=simple
ExecStartPre=/usr/bin/ip link set ens33 promisc on
ExecStart=/usr/local/bin/snort \
    -c /etc/snort/snort.lua \
    -i ens33 \
    -l /var/log/snort \
    -A alert_fast \
    -s 65535 \
    -k none \
    --daq afpacket
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
User=root

[Install]
WantedBy=multi-user.target
```

### 2. Enable & Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable snort3
sudo systemctl start snort3
sudo systemctl status snort3
```

### 3. Monitor Logs via Journald

```bash
sudo journalctl -u snort3 -f
```

---

## Log Analysis

### Alert Log Location

```
/var/log/snort/alert_fast.txt
```

### Sample Alert Output

```
05/20-14:32:07.456321 [**] [1:1000001:1] "ICMP Ping Detected" [**] [Priority: 0] {ICMP} 192.168.1.50 -> 192.168.1.1
05/20-14:32:09.123456 [**] [1:1000003:1] "Possible Nmap SYN Scan" [**] [Priority: 0] {TCP} 10.0.0.5:51234 -> 192.168.1.10:80
```

### Parse Alerts with Grep

```bash
# Show all unique alert signatures
grep "\[\*\*\]" /var/log/snort/alert_fast.txt | awk -F'"' '{print $2}' | sort | uniq -c | sort -rn

# Filter by source IP
grep "192.168.1.50" /var/log/snort/alert_fast.txt

# Count alerts per minute
grep "14:3" /var/log/snort/alert_fast.txt | wc -l

# Show SSH brute force hits
grep "SSH Brute Force" /var/log/snort/alert_fast.txt
```

### Live Tail Alerts

```bash
tail -f /var/log/snort/alert_fast.txt
```

---

## Verification & Testing

### Test 1: Trigger ICMP Rule

From an attacker or test machine:

```bash
ping -c 5 192.168.1.1
```

Expected alert:

```
[**] [1:1000001:1] "ICMP Ping Detected" [**]
```

### Test 2: Trigger SYN Scan Rule (Nmap)

```bash
nmap -sS 192.168.1.1 -p 1-1000
```

Expected alert:

```
[**] [1:1000003:1] "Possible Nmap SYN Scan" [**]
```

### Test 3: Trigger SQL Injection Rule

```bash
curl "http://192.168.1.10/index.php?id=1%20UNION%20SELECT%201,2,3"
```

Expected alert:

```
[**] [1:1000006:1] "Possible SQL Injection - UNION SELECT" [**]
```

### Test 4: Use a Known Malicious PCAP

```bash
# Download a known malicious PCAP from Malware Traffic Analysis
wget https://example.com/malware-sample.pcap -O /tmp/test.pcap
sudo snort -c /etc/snort/snort.lua -r /tmp/test.pcap -A alert_fast
```

---

## Troubleshooting

| Issue | Cause | Resolution |
|---|---|---|
| `ERROR: Unable to find plugin: afpacket` | LibDAQ not installed correctly | Reinstall libdaq and run `sudo ldconfig` |
| `snort: error while loading shared libraries` | Missing shared libs | Run `sudo ldconfig` after install |
| No alerts generated | Promiscuous mode off | `sudo ip link set ens33 promisc on` |
| `PCRE2 not found` during build | Missing dev package | `sudo apt install libpcre2-dev` |
| High CPU usage | Snaplen too high or fast NIC | Reduce snaplen or tune buffer_size |
| `Permission denied` on `/var/log/snort` | Wrong ownership | `sudo chown -R snort:snort /var/log/snort` |
| Config validation fails | Syntax error in .lua or rules file | Run `snort -c snort.lua -T` and review error output |

---

## Security Hardening Notes

- **Dedicated User:** Run Snort as a non-root service user after initial capability assignment:

```bash
sudo useradd -r -s /sbin/nologin snort
sudo chown -R snort:snort /var/log/snort /etc/snort
sudo setcap cap_net_raw,cap_net_admin=eip /usr/local/bin/snort
```

- **Separate Management VLAN:** Place Snort's management interface on an isolated VLAN to prevent tampering.
- **Log Integrity:** Forward logs to a remote SIEM (Splunk, ELK, Wazuh) immediately — don't rely solely on local logs.
- **Rule Updates:** Integrate with `pulledpork3` or `snort-update` to pull the latest Talos community ruleset daily.
- **Disable Unnecessary Preprocessors:** Reduce attack surface by commenting out unused inspectors in snort.lua.
- **Alert Tuning:** Whitelist known-good traffic to reduce false positives — tune `threshold` and `suppress` directives.

---

## Summary & Findings

### What Was Accomplished

| Task | Status |
|---|---|
| System preparation & dependency install | ✅ Complete |
| LibDAQ compiled and installed | ✅ Complete |
| Snort 3 compiled from source | ✅ Complete |
| snort.lua base configuration | ✅ Complete |
| HOME_NET defined and validated | ✅ Complete |
| Local rule file created (10 rules) | ✅ Complete |
| ICMP, TCP, HTTP, DNS, FTP detection rules | ✅ Complete |
| Passive IDS mode verified | ✅ Complete |
| Live traffic testing (ping, nmap, curl) | ✅ Complete |
| Systemd service deployment | ✅ Complete |
| Log analysis and alert verification | ✅ Complete |

### Key Observations

- Snort 3 uses **Lua** for configuration (major change from Snort 2's preprocessor `.conf` syntax).
- The `afpacket` DAQ is optimal for Linux-based passive monitoring — zero-copy packet access.
- Hyperscan integration significantly improves regex-heavy rule matching performance.
- Promiscuous mode is **mandatory** for monitoring traffic not destined to the Snort host.
- On virtual machines, use `-k none` to disable checksum errors caused by hardware offloading.

### Recommendations

1. Integrate with **Barnyard2** or **u2json** for Unified2 log forwarding to a SIEM.
2. Subscribe to **Talos Snort Community Rules** for coverage of CVEs and known threats.
3. Set up **PulledPork3** for automated rule management and updates.
4. Consider **Snort in IPS mode** (inline with NFQUEUE or bridge interface) for active blocking once tuning is complete.
5. Deploy **multiple sensors** at ingress/egress points and segment by traffic zone.

---

*End of Report — Snort IDS Installation & Configuration*  
*Document generated on: 2026-05-20*