# NETWORK Forensics

## Overview

Network traffic investigation is the process of capturing, recording, and analyzing data packets traveling across a network in order to understand communication behavior, detect anomalies, and support security or forensic objectives.

> **📌 Expert Perspective:** For a Blue Team engineer or digital forensic analyst, network traffic analysis is not merely looking at raw packets — it is the art of reconstructing the full story of what happened on the wire.

---

## 1. Overview of Core Concepts

### 1.1. IP Address

* **Description:** An IP address (Internet Protocol Address) is the unique identifier of a device on a network. Just like a house number in real life, every computer, phone, or server connected to the Internet is assigned an IP address so that other devices know where to send data.
* **Versions:** There are currently two common versions: **IPv4** (e.g., `192.168.1.1`) and **IPv6** (a longer format, e.g., `2001:0db8::1`).
* **Forensic Value:** When investigating network traffic, the IP address helps identify the source and destination of each packet, allowing analysts to map communication between hosts.

### 1.2. Protocol

* **Description:** A protocol is a set of predefined rules and formats that allow entities on a network to communicate with each other. In real life, if two people speak different languages, they cannot communicate easily; they must follow a common “protocol” (same language, taking turns speaking, signaling when finished).
* **Network Context:** Apple and Microsoft operating systems cannot “talk” directly to each other without protocols that define how information is transmitted. Common protocols include TCP, UDP, HTTP, HTTPS, DNS, FTP, and others.
* **Forensic Value:** Each protocol handles a specific task in the data transmission process. Understanding which protocol is being used is essential for correctly interpreting packet content.

### 1.3. Port

* **Description:** A port is essentially a specific “doorway” into a computer or server.
* **Analogy:** Imagine a computer as a large building with a single unique address (the IP address). Inside this building are many departments, each with its own numbered door for handling different tasks.
* **Common Ports:**
  * **Port 80** → handles regular web traffic (HTTP)
  * **Port 443** → handles secure web traffic (HTTPS)
  * **Port 53** → used for DNS (domain name resolution)
  * **Port 22** → used for SSH (secure remote access)
* **Forensic Value:** Thanks to this clear port classification system, a single computer can handle thousands of tasks at the same time without mixing up data between different applications. Identifying unusual ports is a common technique in threat detection.

---

## 2. Network Traffic Investigation Process

### 2.1. Capturing Network Traffic

* **Description:** Capturing network traffic means recording all data that passes through the network — also known as network logs. Unlike web logs or browser logs (which only record information from a specific application, like a camera watching just one small shop), network logs record **everything** that travels over the network to the ports of a computer or server.
* **Limitation:** Ordinary computers do not automatically save network logs because they would quickly run out of memory. To obtain network logs, you must actively install monitoring software (like “hiring a security guard”).
* **Automatic Sources:** There are two places that usually record automatically: the **router** and the **Internet Service Provider (ISP)**, but access rights are required to view them.
* **Analogy:** Being able to capture network traffic is like installing cameras throughout an entire company — you can see which “intruders” are trying to break in, or track when unusual activity occurs.

**Popular Tools:**

* **Wireshark** — the top tool, with an intuitive interface and deep analysis capabilities.
* **NetworkMiner** — specializes in extracting files and information from network traffic.
* **Brim / Zeek** — strong at log analysis and anomaly detection at large scale.

**Basic Wireshark Usage:**
Open the application → select a network interface (Wi-Fi or Ethernet) → click the start capture button → when finished, click the stop button (square icon) → view and save the log file (`.pcap`).

### 2.2. Analyzing Network Traffic with Wireshark

Network traffic analysis has many approaches, but the most common and useful operations include: viewing the protocol hierarchy, applying filters, examining packet details, following TCP streams, and exporting objects.

#### a. Protocol Hierarchy

* **Description:** This provides a high-level overview: more complex protocols sit at the upper layers (handling higher-level tasks), then pass data down to simpler protocols at lower layers to perform specialized functions. For example, HTTP sits on top of TCP, TCP sits on top of IP, and IP sits on top of Ethernet.
* **How to view:** `Statistics → Protocol Hierarchy`.
* **Forensic Value:** From here you can see which protocols consume the most bandwidth, helping you quickly focus your analysis.

#### b. Display Filters

* **Description:** Filters allow you to show only the packets you care about and remove noise. This is the most important tool when analyzing traffic.
* **Commonly used filters:**
  * `ip.addr == 192.168.1.10` → view traffic related to this IP
  * `tcp.port == 443` → show only HTTPS traffic
  * `http` → show only HTTP packets
  * `dns` → view DNS queries
* **Reference:** Full list of display filters → [https://wiki.wireshark.org/DisplayFilters](https://wiki.wireshark.org/DisplayFilters)

#### c. Packet Details and Hexadecimal View

* **Description:** The Wireshark interface is divided into three main panes:
  * **Top pane (Packet List):** a list of packets, color-coded according to rules (e.g., TCP often has its own color).
  * **Middle pane (Packet Details):** detailed breakdown of each protocol layer of the selected packet.
  * **Bottom pane (Packet Bytes):** raw data in hexadecimal (hex) and ASCII format.

#### d. Follow TCP Stream

* **Description:** This is a very convenient feature — like “opening the chat window” of a specific conversation between two parties.
* **How to use:** Right-click any packet belonging to the stream → select **Follow → TCP Stream**.
* **Forensic Value:** Wireshark will reassemble the entire exchange (request/response) into a readable text block, which is especially useful when analyzing HTTP, SMTP, or other text-based protocols.

---

## 3. Summary

Network traffic investigation starts with understanding three fundamental components:

* **IP address** → who is communicating
* **Protocol** → what language they speak
* **Port** → which room they enter

Then use tools like Wireshark to capture and analyze all data passing through the network, in order to detect anomalies, monitor activity, or support security and investigative purposes.

