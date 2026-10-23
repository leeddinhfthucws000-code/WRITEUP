# Transmission Control Protocol / Internet Protocol (TCP/IP) Suite

## 1. Overview
The **TCP/IP model** was created by Vint Cerf and Bob Kahn and developed in the 1970s. It is a comprehensive protocol suite consisting of various layered sub-protocols with distinct responsibilities. Its primary objective is to enable heterogeneous open systems to communicate seamlessly.

---

## 2. Architecture of the TCP/IP Model
The TCP/IP model comprises **4 distinct layers**, each operating independently:

| Layer | Responsibilities | Corresponding OSI Reference Layer |
| :--- | :--- | :--- |
| **4. Application Layer** | Interacts directly with users via web applications, software, or protocols to transmit, display, and process data. Handles data formatting (e.g., `.docx`, `.png`), cryptographic encryption/decryption, compression, and session management (establishing, synchronizing, and terminating sessions between end-to-end entities). | 7. Application <br> 6. Presentation <br> 5. Session |
| **3. Transport Layer** | Controls end-to-end data transmission, performs data segmentation, manages port addressing, and ensures communication reliability between two devices. | 4. Transport |
| **2. Internet Layer** | Responsible for logical addressing, packet routing, and determining the optimal path for packets across interconnected networks using IP addresses. | 3. Network |
| **1. Network Access Layer** | Encapsulates data into frames, handles local error control, and utilizes MAC addresses for transmission. Transmits raw binary bitstreams (sequences of 0s and 1s) across physical transmission media. | 2. Data Link <br> 1. Physical |

---

## 3. Protocol Suite

### 3.1. Application Layer Protocols

#### DNS (Domain Name System)
* **Concept / Definition:** A hierarchical naming system that resolves domain names.
* **Responsibilities / How it Works:** Because remembering numeric IP addresses for numerous websites on the internet is impractical, DNS translates user-friendly domain names into corresponding IP addresses to route clients to their target destinations.

#### HTTP / HTTPS
* **Concept / Definition:** Application-layer protocols governing the retrieval and transmission of web resources (text, images, media) between servers and clients.
* **Responsibilities / How it Works:** Upon reaching a target web address, the protocol requests specific content, and the web server validates client authorization before fulfilling or denying the request.

---

### 3.2. Transport Layer Protocols

#### TCP (Transmission Control Protocol)
* **Concept / Definition:** A connection-oriented protocol designed to ensure reliable, sequenced, and error-free data delivery between two endpoints.
* **Responsibilities / How it Works:** Its operation involves three main phases:
    1. *Connection Establishment (3-Way Handshake):* Step 1 (SYN) - the sender transmits a connection request packet; Step 2 (SYN-ACK) - the receiver acknowledges and agrees to connect; Step 3 (ACK) - the sender sends a final confirmation, establishing the active session.
    2. *Reliable Data Transfer:* Upper-layer payloads are segmented, tagged with sequence numbers, and injected into the network. The receiver responds with Acknowledgments (ACKs) corresponding to sequence numbers; timeouts trigger automatic packet retransmissions.
    3. *Connection Termination:* Graceful teardown procedures release network system resources upon completion.

#### UDP (User Datagram Protocol)
* **Concept / Definition:** A connectionless protocol that prioritizes high-speed transmission.
* **Responsibilities / How it Works:** Transmits data rapidly without delivery guarantees, error-checking overhead, or prior connection handshakes between endpoints.

---

### 3.3. Internet Layer Protocols

#### IP (Internet Protocol)
* **Concept / Definition:** Connects subnets into an internetwork for data transmission utilizing packet switching.
* **Responsibilities / How it Works:** 
    * *Encapsulation:* Wraps upper-layer segments with an IP header containing source and destination IPs.
    * *Connectionless Routing:* Sends packets independently across different paths without prior path setup.
    * *Best-Effort Delivery:* Transmits data with maximum effort but without absolute guarantees against packet loss, corruption, or out-of-order arrival.

#### ICMP (Internet Control Message Protocol)
* **Concept / Definition:** A diagnostic protocol utilized to evaluate host connectivity and report operational anomalies.
* **Responsibilities / How it Works:** Operations follow a query-response or error notification cycle:
    * *Echo Request / Reply:* Used in network utilities like `ping` to compute latency and packet loss rates.
    * *Error Reporting:* Intermediate routers or destination hosts automatically generate ICMP error messages (e.g., `Destination Unreachable`, `Time Exceeded`) when transmission failures occur.

#### ARP (Address Resolution Protocol)
* **Concept / Definition:** Resolves data link layer physical addresses from network layer logical addresses.
* **Responsibilities / How it Works:** When a host knows a destination IP (e.g., a gateway modem at `192.168.1.1`) but lacks its MAC address required for Ethernet/Wi-Fi framing, ARP broadcasts an ARP Request query across the local subnet. The target device responds directly via an ARP Reply providing its MAC address, which is subsequently cached in the local ARP Table for future communication.
