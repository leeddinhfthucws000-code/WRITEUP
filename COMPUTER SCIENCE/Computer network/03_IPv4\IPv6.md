# Internet Protocol version IPv4 & IPv6 Overview

## 1. Overview
IPv4 and IPv6 are two principal versions of the Internet Protocol (IP). Originally, IPv4 was the most widely deployed version; however, due to IPv4 address exhaustion, IPv6 was introduced as its successor. In current networking environments, both protocols are frequently operated in a dual-stack configuration to ensure performance and facilitate smooth transitions.

---

## 2. IPv4 Structure and Representation
* **Concept / Definition:** An IPv4 address consists of 32 bits, divided into four 8-bit sections known as octets.
* **Responsibilities / How it Works:** Because raw binary representations (0s and 1s) are difficult for humans to process, IPv4 addresses are converted into dotted-decimal notation ranging from 0 to 255 per octet (e.g., `172.16.30.56`).

---

## 3. IPv4 Classification

### 3.1. Classification by Functionality

#### Broadcast
* **Concept / Definition:** A transmission method where a host sends a packet to a broadcast address, which routers and switches replicate and deliver to all devices within the same Local Area Network (LAN).
* **Responsibilities / How it Works:** The broadcast IP address is typically `255.255.255.255` or a private IP ending with `.255`.

#### Multicast
* **Concept / Definition:** A communication method where data is transmitted from a single source to a specific group of devices that have registered to a Multicast IP group.
* **Responsibilities / How it Works:** Packets are not broadcasted indiscriminately; addresses fall within the range from `224.0.0.0` to `239.255.255.255`.

#### Loopback
* **Concept / Definition:** A specialized IP address used by a host to communicate with itself (self-loopback).
* **Responsibilities / How it Works:** Data sent to `127.0.0.1` never leaves the network interface card or travels across physical network media; any address starting with `127.` designates self-reference.

#### Unicast
* **Concept / Definition:** A one-to-one communication method where a host sends data directly to a single specific destination device.
* **Responsibilities / How it Works:** Unlike Multicast or Loopback, virtually all standard public and private IP addresses encountered in typical networks default to Unicast IPs.

### 3.2. Classification by Operational Scope

#### Public IP
* **Concept / Definition:** Addresses allocated by international network authorities to route traffic globally across the public Internet.
* **Responsibilities / How it Works:** Every publicly accessible device requires a public IP address (e.g., Google’s DNS server `8.8.8.8`).

#### Private IP
* **Concept / Definition:** Addresses reserved exclusively for internal use within private networks (homes, enterprises, educational institutions).
* **Responsibilities / How it Works:** These addresses are non-routable on the public Internet to conserve address space. Common ranges include:
    * *Class A Private:* `10.0.0.0` to `10.255.255.255` (large corporate networks).
    * *Class B Private:* `172.16.0.0` to `172.31.255.255` (medium-sized enterprise networks).
    * *Class C Private:* `192.168.0.0` to `192.168.255.255` (widely used for home Wi-Fi and cafes).

### 3.3. Classful Addressing (Network Classes)
Classful addressing categorizes IPv4 addresses based on network capacity, dividing an IP address into a Network ID and a Host ID.

#### Class A (Large-Scale Organizations)
* **Concept / Definition:** Designed for massive networks, identified by a leading bit of `0` and a first-byte range from `1.0.0.0` to `126.0.0.0` (with `0.x.x.x` and `127.x.x.x` reserved for special purposes).
* **Responsibilities / How it Works:** Uses the first 8 bits for the Network ID and the remaining 24 bits for the Host ID, supporting few networks (126 networks) but an immense number of hosts (over 16.7 million per network).

#### Class B (Medium and Large Enterprises)
* **Concept / Definition:** Designed for medium-to-large organizations, identified by a leading bit sequence of `10` and a first-byte range from `128.0.0.0` to `191.255.0.0`.
* **Responsibilities / How it Works:** Allocates 16 bits for the Network ID and 16 bits for the Host ID, supporting approximately 16,384 networks with up to 65,534 hosts each (commonly used in large enterprises and universities).

#### Class C (Small Businesses and Home Networks)
* **Concept / Definition:** Designed for small organizations and home networks, identified by a leading bit sequence of `110` and a first-byte range from `192.0.0.0` to `223.255.255.x`.
* **Responsibilities / How it Works:** Supports millions of small networks, with each network accommodating up to 254 hosts (ideal for small offices, cafes, and home Wi-Fi).

#### Class D (Multicast)
* **Concept / Definition:** Reserved exclusively for multicasting, identified by a leading bit sequence of `1110` and a range from `224.0.0.0` to `239.255.255.255`.
* **Responsibilities / How it Works:** Enables simultaneous data transmission from a single source to a registered group of listening devices rather than individual unicast or network-wide broadcast.

#### Class E (Research and Experimental)
* **Concept / Definition:** Reserved entirely for research, development, experimental networking technologies, or contingency planning.
* **Responsibilities / How it Works:** Identified by a leading bit sequence of `11110` with a range from `240.0.0.0` to `255.255.255.255`.
