# CRYPTOGRAPHY OVERVIEW
 
## 1. Concept
 
### Core Concept
**Cryptography** is a field of study and application of techniques to protect information and data, ensuring **confidentiality, integrity, and authenticity** by transforming information and data into an incomprehensible form to unauthorized parties.
 
---
 
## 2. Information Security Objectives (CIA)
 
* **C - Confidentiality:** Protecting information from unauthorized access or disclosure.
* **I - Integrity:** Ensuring that data is not modified in an unauthorized or improper manner.
* **A - Availability:** Ensuring that systems and data are always accessible and usable by authorized users when needed.
### Two Supplementary Objectives
* **Authentication:** The process of verifying the identity of entities participating in information exchange.
* **Non-repudiation:** The capability to ensure that information exchange operations prevent parties from denying their participation in the exchange process.
---
 
## 3. Key Terminology
 
| Term | Notation | Definition |
| :--- | :--- | :--- |
| **Plaintext** | *m* or *P* | The original data constructed using any predefined alphabet or set of symbols. |
| **Ciphertext** | *c* or *C* | The plaintext after being encrypted and transformed into incomprehensible data. |
| **Cipher** | - | The mathematical algorithm used to transform plaintext into ciphertext. |
| **Encipher (Encryption)** | - | The process of converting plaintext into ciphertext. |
| **Decipher (Decryption)** | - | The process of converting ciphertext back into plaintext. |
| **Key** | *ke* (encryption), *kd* (decryption) | A necessary condition for bidirectional transformation between plaintext and ciphertext. |
| **Cryptanalysis** | - | The science and art of analyzing, breaking, or bypassing the security mechanisms of a cryptographic system to recover the original information (plaintext) without possessing the key. |
 
---
 
## 4. Kerckhoffs's Principle
 
### Core Concept
A cryptographic system must be secure even if an attacker knows the complete inner workings of the encryption algorithm (i.e., procedures and mathematical formulas used for encryption and decryption are publicly known). The security of the system relies entirely on keeping the **Key secret** (held only by the sender and receiver; an attacker with knowledge of the algorithm but without the key remains completely unable to decrypt).
 
### Why It Matters
* **Rejection of 'Security through Obscurity':** Avoids reliance on ambiguity and concealment for protection.
* **Community Vetting:** Publicly disclosed algorithms allow thousands of experts and white-hat hackers to dissect, find, and patch vulnerabilities rapidly, producing robust systems.
* **Mitigation of Self-Made Algorithm Flaws:** Proprietary or hidden algorithms frequently contain fatal flaws due to the lack of peer review and critical scrutiny.
* **Core Security Resides in the Key:** The algorithm structure is like a lock open for public inspection; true security lies strictly in keeping the key secret.
---
 
## 5. Security Attacks According to Standard X.800
 
According to standard **X.800** (an ITU-T standard defining security architecture for the OSI model), security attacks are categorized into two primary groups aligned with the information flow model:
 
| Attack Type | Nature | Detection / Defense Focus | Violates |
| :--- | :--- | :--- | :--- |
| **Passive Attacks** | Eavesdropping or monitoring transmitted information without modifying content or system state. | Extremely hard to detect → focus on **PREVENTION** (e.g., strong encryption). | Confidentiality |
| **Active Attacks** | Modifying data streams, interfering with systems, or creating fabricated data. | Easier to detect but hard to fully prevent → focus on **DETECTION & RECOVERY**. | Integrity, Availability, Authentication |
 
### 5.1 Passive Attacks
* **Nature:** Eavesdropping or monitoring information transmitted across the network. Attackers merely observe and collect data without modifying content or system states.
* **Characteristics:** Extremely difficult to detect due to the absence of noticeable system disruptions or data alterations. Consequently, defense focuses strictly on **PREVENTION** (e.g., strong encryption so intercepted data remains unreadable).
* **Examples & Context:** Release of message contents and Traffic analysis (analyzing communication parties and frequency even when payloads are encrypted). Corresponds to **Interception** in information flow models, directly compromising **Confidentiality**.
### 5.2 Active Attacks
* **Nature:** Actions involving modification of data streams, direct system interference, or creation of fabricated data.
* **Characteristics:** Easier to detect than passive attacks (as they leave system error traces or data corruption), but difficult to prevent completely. Defense focuses on **DETECTION & RECOVERY**.
* **Modification & Interruption:** Attackers disrupt communication flows (impairing Availability) or alter transmitted message contents (violating Integrity and Authentication). A classic example is **Denial of Service (DoS)**.
* **Fabrication (Masquerade / Replay):** Attackers generate unauthorized fake data, spoof sender identities, or replay captured valid packets to deceive destination systems. Severely violates **Integrity & Authentication**.
---
 
## 6. Cryptosystems
 
A cryptosystem is a comprehensive framework describing the complete cycle of encrypting plaintext into ciphertext and decrypting ciphertext back into plaintext.
 
### 6.1 Symmetric Encryption (Private-Key Cryptography)
* **Principle:** Sender and receiver share a single secret key for both encryption and decryption.
* **Characteristics:** High processing speed, highly efficient for bulk data encryption.
* **Limitations:** Presents significant challenges regarding secure secret key distribution and management across public networks.
* **Typical Examples:** DES, 3DES, AES, RC4, Blowfish.
> **Analogy:** Imagine you have data 'a' to share with a classmate. You both pre-agree on a shared key during class. At home, you encrypt data 'a' using that key and send it via email. Upon receipt, your friend uses the exact same key to decrypt it.
 
### 6.2 Asymmetric Cryptography (Public-Key Cryptography)
* **Principle:** Each entity possesses a mathematically linked key pair: a Public Key (used for encryption/locking) and a Private Key (used for decryption/unlocking, kept strictly secret).
* **Dual Application:** Provides confidentiality (via receiver's public key) and supports digital signatures and authentication (via sender's private key).
* **Characteristics:** Significantly slower than symmetric encryption; primarily utilized for session key exchanges or digital signatures rather than direct bulk data encryption.
* **Typical Examples & Tools:** RSA, ECC, Diffie-Hellman; generated via complex one-way mathematical functions using tools like OpenSSL or cryptographic programming libraries.
> **Analogy:** To receive confidential information 'A' from a friend, you generate a key pair (Key B and Key C) using a cryptographic tool. They are mathematically linked such that data encrypted with Key B can only be decrypted by Key C. You publish Key B publicly. Your friend encrypts data 'A' using Key B and sends it to you; you then decrypt it using your private Key C.
 
### 6.3 Hybrid Cryptosystem
Combines the strengths of both paradigms: **Asymmetric cryptography** is used to securely exchange a temporary session key, and **Symmetric encryption** is used to encrypt the actual payload efficiently.
 
| Paradigm | Speed | Key Distribution | Typical Role in Hybrid |
| :--- | :--- | :--- | :--- |
| **Symmetric** | Fast | Hard (shared secret) | Encrypts the actual payload |
| **Asymmetric** | Slow | Easy (public key) | Securely exchanges the session key |
 

