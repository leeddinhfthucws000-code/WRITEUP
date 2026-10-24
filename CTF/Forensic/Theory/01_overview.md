# DIGITAL FORENSICS OVERVIEW

---

## 1. What is Digital Forensics and What is it for?

### Core Concept
Digital Forensics is the practice of digital investigation. Its primary mission is to analyze data and files to uncover hidden traces or left-behind artifacts to determine what happened. Whether dealing with a file, a computer, an image, or a log entry, digital forensics extracts secret information from network activity, access logs, search history, and digital storage media (such as hard drives and mobile devices) to identify admissible evidence.

### Purpose & Applications
* **In CTF Competitions:** Forensics involves analyzing and tracking digital traces to discover the hidden **Flag**.
* **In Real-World Scenarios:** Forensics is utilized for investigating cyberattacks, Incident Response (DFIR), recovering lost/deleted data, digital criminal investigation, resolving legal disputes, and gathering legally sound evidence.

---

## 2. Core Branches in Digital Forensics

To master Digital Forensics, it is essential to thoroughly understand these 5 specialized branches:

| Branch | Focus & Description |
| :--- | :--- |
| **File Analysis & Steganography** | Analyzing file structures (Header/Magic Bytes), recovering deleted files, or extracting data hidden within images, audio, video, and documents. |
| **Memory Forensics** | Analyzing volatile memory (RAM dumps) to identify running malicious processes, covert network connections, passwords stored in RAM, or fileless malware. |
| **Disk & File System Forensics** | Analyzing OS artifacts across Windows, Linux, and macOS (Registry, Event Logs, Master File Table - MFT, Prefetch, Shellbags, Journaling). |
| **Network Forensics** | Analyzing packet capture files (`.pcap`, `.pcapng`), tracking network traffic, and detecting port scans, vulnerability exploits, or unauthorized file transfers. |
| **Log Analysis** | Parsing and tracing logs from Web Servers (Apache, Nginx), Operating Systems, Firewalls, and Syslog to reconstruct the attacker's activity timeline. |

---

## 3. Standard Digital Forensics Workflow (DFIR)

A standardized investigation workflow ensures evidence integrity and optimizes analysis time:

1. **Identification:** Define the target, scope of investigation, and the types of data to collect (image files, pcap, memory dumps, logs).
2. **Preservation:** Ensure original data remains untampered. Create forensic disk images (`dd`/image formats) and calculate Hash values (MD5/SHA256) to verify data integrity.
3. **Analysis:** Utilize specialized forensic tools to extract artifacts, decrypt data, search for hidden strings/keywords, and reconstruct the incident.
4. **Reporting / Flag Extraction:** Compile event timelines, draw conclusions regarding the root cause of the incident, or successfully retrieve the flag in CTF challenges.

---

## 4. Tools in Digital Forensics

> **Concept of Tools:** A tool is any software or utility that makes the investigation faster and more precise. In Digital Forensics, the **Linux Terminal** is an extremely common and powerful working environment.
