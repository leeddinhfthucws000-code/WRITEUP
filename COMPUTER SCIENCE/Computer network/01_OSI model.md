# Open Systems Interconnection (OSI) Model

## 1. What is the OSI Model?
The **OSI (Open Systems Interconnection) model** is a conceptual framework published by the **International Organization for Standardization (ISO)** in 1984. Its initial purpose was to enable open systems—including various operating systems, web applications, and network devices—to communicate with one another seamlessly. 

Over time, more stable and practical protocols (such as TCP/IP) have largely replaced it in real-world deployments. Consequently, the primary application of the OSI model today is for educational, research, and analytical purposes.

---

## 2. Structure of the OSI Model
The OSI model consists of **7 distinct layers** (structured from Layer 7 down to Layer 1). Each layer performs a specific, dedicated task and communicates exclusively with the layers immediately above and below it.

| Layer | Name | Primary Responsibilities |
| :---: | :--- | :--- |
| **7** | **Application** | Interacts directly with end-users via web applications, software, or protocols to transmit, receive, and display data. |
| **6** | **Presentation** | Handles data format translation (e.g., `.docx`, `.png`, `.mp4`), data encryption/decryption, and compression to ensure data can be properly understood across different devices. |
| **5** | **Session** | Establishes, manages, synchronizes, and terminates communication sessions (transactions) between end-to-end entities. |
| **4** | **Transport** | Manages end-to-end data delivery, performs segmentation, manages port addressing, and ensures reliability for communication between two devices. |
| **3** | **Network** | Handles routing and determines the optimal path for packets across different networks using logical addressing (IP addresses). |
| **2** | **Data Link** | Encapsulates data into frames, performs local error control, and uses physical addresses (MAC addresses) to transmit data. |
| **1** | **Physical** | Transmits raw binary bitstreams (sequences of 0s and 1s) across the physical transmission media. |

> **Note:**
> * **Layers 7, 6, 5, 4** operate on the source and destination hosts to process and handle application data.
> * **Layers 3, 2, 1** operate within the network infrastructure responsible for data transmission.

---

## 3. Protocol Data Units (PDU)

* **Layers 7, 6, 5 (Application, Presentation, Session):** The PDU is generically referred to as **Data** (Raw Data).
* **Layer 4 (Transport):** The PDU is called a **Segment** (if using TCP) or a **Datagram** (if using UDP).
* **Layer 3 (Network):** The PDU is called a **Packet**.
* **Layer 2 (Data Link):** The PDU is called a **Frame**.
* **Layer 1 (Physical):** The PDU is called **Bits** (Binary stream of 0s and 1s).

---

## 4. Data Transmission Process in the OSI Model
Data transmission in the OSI model follows a two-way process: 
* **Encapsulation** at the sender device (from Layer 7 down to Layer 1).
* **Decapsulation** at the receiver device (from Layer 1 up to Layer 7).

### 4.1. Encapsulation Process (Sender Side)

| Layer | Data Transformation |
| :--- | :--- |
| **Application Layer** | Data remains in its original form (raw application payload). No headers or trailers are attached yet. |
| **Presentation Layer** | Formats, compresses, or encrypts the application data as required. |
| **Session Layer** | Establishes session management context for data transfer. |
| **Transport Layer** | Data is segmented and appended with a **Transport Header** (containing source/destination Port numbers and flow control parameters): <br> `[ Transport Header ] + [ Data ]` |
| **Network Layer** | Receives the segment from Layer 4 and prepends a **Network Header** (containing source and destination IP addresses): <br> `[ IP Header ] + [ Transport Header ] + [ Data ]` |
| **Data Link Layer** | Receives the packet from Layer 3, prepends a **MAC Header** (containing source/destination MAC addresses), and appends a **Frame Trailer** (FCS) for transmission error checking: <br> `[ MAC Header ] + [ IP Header ] + [ Transport Header ] + [ Data ] + [ Trailer ]` |
| **Physical Layer** | The entire binary Frame structure (0s and 1s) is converted into purely physical signals (electrical pulses, light signals, or radio waves) for transmission over media. |

---

### 4.2. Decapsulation Process (Receiver Side)

| Layer | Data Transformation |
| :--- | :--- |
| **Physical Layer** | Receives physical signals, converts them back into a bitstream, reconstructs the Frame structure, and forwards it to Layer 2. |
| **Data Link Layer** | Reads the MAC Header to verify if the frame is intended for its Network Interface Card (NIC), checks for errors using the Frame Trailer, strips L2 headers, and forwards to Layer 3: <br> `[ IP Header ] + [ Transport Header ] + [ Data ]` |
| **Network Layer** | Reads the IP Header to verify the destination IP address, strips the L3 header, and forwards the payload to Layer 4: <br> `[ Transport Header ] + [ Data ]` |
| **Transport Layer** | Reads the Transport Header, verifies the destination Port number, reorders segments if necessary (when using TCP), strips L4 header, and passes to Layer 5: <br> `[ Data ]` |
| **Session Layer** | Handles session termination and synchronizes the incoming data stream. |
| **Presentation Layer** | Decrypts, decompresses, and translates data into a standardized format. |
| **Application Layer** | Displays the fully processed and restored data directly to the receiving user's application. |
