# COMPREHENSIVE LOG ANALYSIS HANDBOOK (Overview Knowledge, Detailed Investigation Techniques, and Hands-on SOC/DFIR Guide)
 
## Overview
 
Logs (system log records) are real-time records capturing all activities, events, transactions, and states occurring across operating systems, applications, network devices, or firewalls. In Security Operations Center (SOC) monitoring, Digital Forensics, and Incident Response (DFIR), logs serve as the "black box" recording the complete digital footprint left behind by users, processes, or attackers.
 
> **📌 Expert Perspective:** For a Blue Team engineer, Log Analysis is not merely reading raw text files; it is the process of correlating information, detecting anomalous behavior, identifying attack vectors, and reconstructing incident timelines.
 
---
 
## 1. Overview of Log Analysis and System Architecture
 
### 1.1. What is a Log and the Role of Log Analysis
 
* **Description:** Logs (system log records) are real-time records capturing all activities, events, transactions, and states occurring across operating systems, applications, network devices, or firewalls.
* **Forensic Value:** In SOC monitoring, Digital Forensics, and Incident Response (DFIR), logs serve as the "black box" recording the complete digital footprint left behind by users, processes, or attackers.
### 1.2. Basic Two-Tier Defense Model: Web Server Log vs. WAF Log
 
* **Description:** In web application infrastructure, monitoring systems primarily rely on two core logging layers:
  * **Web Server Logs (Access & Error Logs):** Analogous to the "shopkeeper's ledger." Every request sent to the web server (viewing pages, clicking buttons, downloading files) is automatically logged into `access.log` (containing client IP, timestamp, URL, HTTP status code). If technical issues or application errors occur, details are aggregated into `error.log`. The web server logs all incoming requests regardless of whether they are legitimate or malicious.
  * **Web Application Firewall Logs (WAF Logs):** Analogous to the "professional security guard's logbook." A Web Application Firewall (WAF, such as ModSecurity) resides directly in front of the Web Server to inspect traffic before it reaches the application. When detecting malware, exploit commands, or malicious payloads, the WAF proactively blocks access and logs granular attack structures into `modsec_audit.log`.
### 1.3. Log Lifecycle Pipeline
 
* **Description:** An enterprise-grade log handling workflow consists of 5 closed-loop steps:
  * **Step 1 - Generation:** Operating systems, applications, WAFs, and firewalls generate event records across various formats (Plaintext, JSON, Syslog, XML, EVTX).
  * **Step 2 - Collection:** Deploying agents (Elastic Agent, Beats, Fluentd, Syslog-ng) to collect real-time logs from endpoints/servers to a central repository.
  * **Step 3 - Parsing & Normalization:** Transforming heterogeneous raw data into a standardized Common Schema for cross-source querying.
  * **Step 4 - Storage & Indexing:** Storing logs in time-series optimized databases (Elasticsearch, Splunk Index, ClickHouse).
  * **Step 5 - Correlation & Analysis:** Applying Sigma Rules, correlation logic, or AI/ML to detect anomalies and trigger security alerts.
---
 
## 2. Enterprise Critical Log Sources Map
 
### 2.1. Windows Event Logs (.EVTX)
 
* **Description:** Windows manages system events in binary `.EVTX` format. The most critical Event IDs include:
| **Event ID** | **Event Name** | **Significance & Investigation Focus** |
|:---|:---|:---|
| 4624 | Successful Logon | Successful authentication. Inspect Logon Type:<br>• Type 2: Interactive (Keyboard/Local)<br>• Type 3: Network (SMB/Pass-the-Hash)<br>• Type 10: RemoteInteractive (RDP) |
| 4625 | Failed Logon | Failed authentication. Clear indicator of Brute-Force or Credential Stuffing attacks. |
| 4688 | Process Creation | A new process has been created. Enable Command Line Auditing to inspect process arguments (e.g., `powershell -enc`, `certutil`). |
| 7045 | Service Installation | A new Windows service was installed. Common Persistence technique utilized by malware. |
 
### 2.2. Linux System Logs & Audit Framework
 
* **Description:** Key log sources on Linux:
  * **Auth Log:** `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL/CentOS): Logs SSH authentication, sudo executions, and password changes.
  * **System Log:** `/var/log/syslog` or `/var/log/messages`: General logs capturing kernel messages and system daemons.
  * **Auditd Framework:** Kernel-level auditing framework tracking system calls (syscalls), sensitive file read/write operations (`/etc/passwd`, `/etc/shadow`), and privilege modifications.
### 2.3. Web Server Access Log (Apache / Nginx)
 
* **Description:** Standard structure of an Access Log entry (Common / Combined Log Format):
```text
[Source IP] - - [Timestamp] "[Method] [URL Path] [Protocol]" [HTTP Status Code] [Response Size (Bytes)] "[Referer]" "[User-Agent]"
```
 
* **Real-world example:** Analyzing a command injection log line:
```text
172.17.0.1 - - [21/Jun/2026:15:30:22 +0700] "POST /ping.php HTTP/1.1" 200 1452 "http://127.0.0.1/" "Mozilla/5.0..."
Payload body: ip=127.0.0.1%3B+whoami%3B+cat+%2Fetc%2Fpasswd
```
 
  * **Source IP:** `172.17.0.1`: Attacker's source IP address.
  * **Timestamp:** `[21/Jun/2026:15:30:22 +0700]`: Exact timestamp of the request.
  * **Request:** `POST /ping.php`: HTTP POST method sending data to `ping.php`.
  * **HTTP Status Code:** `200 (OK)`: Request was successfully executed by the web server.
  * **Response Size:** `1452 Bytes`: Abnormally large response size (compared to typical hundreds of bytes), indicating sensitive system data (`/etc/passwd`) was exfiltrated.
  * **Payload:** `ip=127.0.0.1%3B+whoami...`: URL Encoded string decoding to `ip=127.0.0.1; whoami; cat /etc/passwd` (`%3B` represents the semicolon `;` command delimiter in Linux).
### 2.4. WAF / ModSecurity Audit Log
 
* **Description:** ModSecurity logs detailed audit entries organized into sections from `--A--` to `--Z--`:
  * **Section A:** `--A--` (Header): Timestamp, Source IP/Port, Destination IP/Port.
  * **Section B:** `--B--` (Request Header): HTTP Request headers (Host, User-Agent, Accept).
  * **Section C:** `--C--` (Request Body): Raw payload body containing malicious input before processing by source code.
  * **Section H:** `--H--` (Audit Trailer): WAF action and rule details, including Pattern Match (Signature), triggered OWASP CRS rules (e.g., `REQUEST-932-APPLICATION-ATTACK-RCE.conf`), Severity level (`Severity: CRITICAL`), and Action code (`Access denied with code 403`).
---
 
## 3. Detailed Investigation of Web Attacks via Logs
 
### 3.1. Path Traversal Attack
 
* **Description:** Attackers inject path traversal sequences (such as `../`, `..\`, or URL Encoded `%2e%2e%2f`) to access files outside the web root directory (e.g., `/etc/passwd`, `c:\windows\win.ini`).
* **Access Log Artifacts:** Repetitive requests containing directory traversal parameters like `../view.php?image=../../../../etc/passwd`.
* **WAF Error Log Artifacts:** OWASP CRS Rule 930 violation alerts (OS File Access Attempt / LFI). Matched phrase: `"etc/passwd"`.
### 3.2. Command Injection / RCE Attack
 
* **Description:** Attackers exploit unsanitized input fields passed directly into system shell commands to execute arbitrary code.
* **Access Log Artifacts:** Presence of command delimiters (`;`, `&&`, `|`, `$`, `` ` ``) combined with common OS utility commands (`whoami`, `cat`, `id`, `wget`, `curl`, `net user`).
* **OS System Log Artifacts:** Unusual child processes spawned by web server users (`www-data`, `nginx`), such as `/bin/sh`, `/bin/bash`, `nc`, or `python -c`.
### 3.3. SQL Injection (SQLi) Attack
 
* **Description:** Attackers insert SQL code fragments into input parameters to manipulate database queries, extract data, or gain unauthorized control.
* **Access Log Artifacts:** Contains SQL keywords (`UNION SELECT`, `OR 1=1`, `SLEEP()`, `INFORMATION_SCHEMA`) or User-Agent signatures from automated scanners like `User-Agent: sqlmap/1.6.11#stable`.
---
 
## 4. Building a Lab Environment and Monitoring Setup
 
### 4.1. Layer 1 Defense Setup: Apache Web Server (XAMPP on Windows)
 
* **Description:** Steps:
  * **1. Install XAMPP:** Download and install XAMPP for Windows at `C:\xampp`.
  * **2. Deploy Code:** Deploy web application files into `C:\xampp\htdocs\` (Web Root Directory).
  * **3. Enable Apache:** Launch XAMPP Control Panel and click **Start** on Apache. Apache automatically starts logging access records to `C:\xampp\apache\logs\access.log`.
### 4.2. Layer 2 Defense Setup: ModSecurity WAF
 
* **Description:** Steps:
  * **1. Add WAF Module:** Download ModSecurity module for Apache Windows (`mod_security2.so`) and place it in `C:\xampp\apache\modules\`.
  * **2. Configure Apache:** Open `C:\xampp\apache\conf\httpd.conf` in Notepad and add the activation directive:
```apache
LoadModule security2_module modules/mod_security2.so
```
 
  * **3. Load OWASP CRS Rules:** Load OWASP Core Rule Set (CRS) into Apache's configuration files.
  * **4. Restart & Verify:** Restart Apache. The detailed audit log file `C:\xampp\apache\logs\modsec_audit.log` will be automatically created.
### 4.3. Real-Time Log Monitoring Using Kali Linux (WSL)
 
* **Description:** Open Kali WSL Terminal with Administrator privileges to monitor logs directly:
```bash
# Monitor Access Log in real time as incoming requests hit the Web Server
tail -f /mnt/c/xampp/apache/logs/access.log
 
# Inspect ModSecurity WAF detailed audit log
cat /mnt/c/xampp/apache/logs/modsec_audit.log
```
 
---
 
## 5. Investigation Workflow and Industry Tool Stack
 
### 5.1. Core Analysis Mindset
 
* **Description:** Four core principles:
  * **1. Baseline Analysis:** Establish normal operational behavior. Any deviation from the baseline warrants investigation.
  * **2. Frequency Analysis:** Analyze event frequency. Look for rare events or sudden high-volume spikes.
  * **3. Stacking / Grouping:** Group log entries by IP address, User, or Process to scope incident impact.
  * **4. Timeline Reconstruction:** Chronologically order events to reconstruct attack paths aligned with Cyber Kill Chain / MITRE ATT&CK.
### 5.2. Practical CLI Commands for Investigation (Linux / Bash)
 
* **Description:** Commonly used commands:
```bash
# 1. Search Access Log for requests containing RCE command delimiters
sudo egrep -i "(\||;|&&|\$)" /var/log/apache2/access.log
 
# 2. Search for sensitive OS system command keywords
sudo egrep -i "(whoami|id|cat|passwd|shadow|wget|curl|uname|sh|bash)" /var/log/apache2/access.log
 
# 3. Inspect Linux authentication logs for privilege escalation (Sudo)
sudo grep -i "sudo" /var/log/auth.log
sudo grep "www-data" /var/log/syslog
 
# 4. Check Bash command history for compromised user accounts
sudo cat /home/compromised_user/.bash_history
sudo cat /root/.bash_history
 
# 5. Check active network connections for established Reverse Shells
sudo netstat -antp
# Or modern equivalent:
sudo ss -antp
 
# 6. Inspect process tree for anomalous spawned child processes
ps auxf
```
 
### 5.3. 4-Step Rapid Incident Response Checklist for SOC Analysts
 
* **Step 1 - Live Forensics Inspection:** SSH into the suspect server immediately. Execute `netstat -antp` and `ps auxf` to verify active attacker presence or outbound Reverse Shells. Isolate the server if necessary.
* **Step 2 - External Perimeter Investigation (Access Log):** Inspect `access.log` filtered by suspect IP to analyze incoming requests and payloads.
* **Step 3 - Determine Success Level:** Correlate HTTP Status Codes and Response Sizes. Status 200 + large response size → Attack Succeeded. Status 403/406 → Blocked by WAF.
* **Step 4 - Scope Internal Compromise (OS Auditing):** Examine `auth.log`, `syslog`, and `.bash_history` to determine if root privileges were obtained.
### 5.4. Industry Standard Tool Stack
 
|**Tool Category**|**Popular Tools**|**Practical Application**|
|:---|:---|:---|
| **SIEM & Log Management** | Splunk, ELK Stack (Elasticsearch, Logstash, Kibana), Wazuh, Graylog | Centralized log aggregation, event correlation, dashboard visualization, and automated alert triggering. |
| **CLI Analysis Tools** | `grep`, `egrep`, `awk`, `sed`, `jq`, `sort`, `uniq`, `journalctl` | Rapid filtering, data extraction, JSON formatting, and real-time terminal querying on Linux. |
| **Windows Forensics** | Eric Zimmerman's Tools (EvtxECmd), Hayabusa | High-speed EVTX log parsing to CSV/Timeline, threat hunting powered by Sigma Rules. |
