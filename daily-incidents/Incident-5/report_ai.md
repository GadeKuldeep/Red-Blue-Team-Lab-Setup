# 🛡 Incident Ticket: SOC-2026-0414-001

**Severity:** High  
**Status:** Escalated  
**Created By:** SOC L1  
**Assigned To:** Incident Response Team  
**Date/Time:** 14 Apr 2026, 11:56 IST  

---

## 📌 Summary
High-volume alerts detected indicating reconnaissance (directory brute-force) followed by SQL injection attempts against DVWA application.

---

## 🌐 Affected Assets
- Target: 192.168.1.16 (Web Server - DVWA)
- Source: 192.168.1.15 (Internal Host)

---

## 🔍 Detection Details
- Total Alerts: 8761  
- Alert Level: 10–11  
- Rule Groups:
  - web, web_scan, recon  
  - web, accesslog, attack, sqlinjection  

---

## 🧠 Analysis
Observed multiple HTTP 400/404 responses consistent with directory brute-force activity (likely Gobuster).  
Subsequently, SQL injection payloads detected:

/dvwa/vulnerabilities/sqli/?id=' UNION SELECT ...


Indicates transition from reconnaissance to exploitation phase.

---

## 📂 Evidence
- Log Source: `/var/log/apache2/access.log`  
- Log Source: `/var/log/dpkg.log`  
- Repeated failed requests (400/404)  
- SQLi payload patterns detected  

---

## 🛡 Actions Taken
- Source IP (192.168.1.15) temporarily isolated  
- Logs correlated across multiple sources  
- Activity monitored for further escalation  

---

## ⚠️ Impact Assessment
- Activity originated from internal network  
- Target application (DVWA) is intentionally vulnerable  
- No confirmed system compromise  

---

## ✅ Conclusion
Activity confirmed as **authorized penetration testing** conducted from internal host.  
No malicious impact observed.

---

## 📌 Recommendations
- Whitelist authorized testing IPs  
- Restrict access to sensitive files (.bak, config)  
- Improve alert tuning for lab environment  

---

## 🔄 Next Steps
- Close ticket OR mark as “Authorized Activity”  
- Inform relevant stakeholders if required  