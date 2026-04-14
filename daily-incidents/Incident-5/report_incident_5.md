To Incident Responder,
From SOC-L1,
Date: 14 Apr 2026
Sub: Level 10,11 alerts occurred.
Time:
Starting => 11:56:00
Ending => 12:05:56
Total alerts are 8761.
IP:
Server: 192.168.1.16
Attacker: 192.168.1.15
Source Locations =>
/apache2/access.log
/dpkg.log
Sir,
Today at 11:00:00 the first alert got generated on the server due to 192.168.1.15.
First normal alerts got generated such as the valid account. Then the web, webscan and the recon
alerts got generated. That made me think about this as suspicious, so I isolated this IP.
And then after some seconds at exactly 12:01:42 the SQL injection related alerts got
generated. Then at last the dpkg, syslog and config_changed like alerts popped up.
The percentages of the alerts according to the alert types =>
Total alerts: 8761
Vulnerability Scanning: 93.02%
Process Injection: 2.3%
Valid Accounts: 1.76%
Sudo and Sudo Catching: 1.67%
Exploit Public Facing Application: 0.67%
File and Directory Discovery: 0.50%
Attacks Performed:
1) Recon
2) SQL injections
Rules that violated:
1) web, web_scan, recon
2) web, accesslog, attack, sqlinjection
Multiple web server 400 error codes from the source attacker IP. That happens only if the host
tries to find any present routes in the URL using brute force.
From the IP 192.168.1.15, that was from the internal subnet. So I thought it was the bigger threat
(because internal threats are more dangerous than external threats),
so I checked all the subnet and I found out that it was the IP of our ethical hacker
who was performing the penetration test.
Thank you,
SOC L1.