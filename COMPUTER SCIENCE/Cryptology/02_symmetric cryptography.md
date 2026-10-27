# SYMMETRIC CRYPTOGRAPHY OVERVIEW
 
## 1. Concept
 
### Core Concept
**Secret-key cryptography** (also called **symmetric cryptography**) is a core pillar of information security. Its defining feature is that a **single identical secret key** is used for both operations:
 
* **Encryption:** Transforming plaintext into ciphertext.
* **Decryption:** Reconstructing ciphertext back to its original plaintext form.
  <img width="1275" height="201" alt="image" src="https://github.com/user-attachments/assets/d0eb2f6b-c843-4d74-8587-74af1f961294" />

---
 
## 2. Six Essential Criteria for a Robust Secret-Key Cryptosystem
 
* **Confusion:** Makes the mathematical relationship between the secret key and the ciphertext extremely complex. The goal is to prevent cryptanalysts from reverse-engineering the key: no matter how much ciphertext an attacker collects, they cannot establish a direct relation to infer key K.
* **Diffusion:** Spreads the influence of each plaintext bit across the entire ciphertext, so that changing a single plaintext bit alters approximately 50% of the ciphertext bits.
* **Sufficiently Long, Random Key:** Protects against brute-force attacks. Currently requires a minimum key length of **128 bits**, generated with strong randomness.
* **Avalanche Effect:** A change in a single character or bit at the input results in a change in approximately half of the output bits.
* **Multi-Round Encryption:** Iteratively applies substitution and permutation operations over multiple rounds (e.g., DES uses 16 rounds, AES uses 10–14 rounds).
* **Public Algorithm (Kerckhoffs's Principle):** The algorithm must be public for global verification and scrutiny; only the **key** is kept secret.
---
 
## 3. Classical Cryptography
 
### Overview
Classical cryptography covers encryption and decryption techniques used from antiquity until the mid-20th century (before electronic computers were developed).
 
### Substitution Techniques
Replaces each character (or group of characters) in the plaintext with another character or symbol according to a fixed rule. Modular arithmetic based on the alphabet size (e.g., `mod 26`) combines two steps:
 
* **Alphabet table:** Contains all allowable characters (e.g., the 26 English letters A–Z, indexed from `0` to `n-1`, where `n` is the total number of characters).
* **Circular shift rule:** Adding or subtracting shift steps during substitution. If the result exceeds the table boundary, the modulo operation (`mod n`) wraps it back to the beginning of the table, so a valid substitution character is always produced without overflow.

| Cipher Name | Operational Mechanism & Characteristics |
| :--- | :--- |
| **Caesar Cipher** | • **Principle:** Shifts each plaintext character by a fixed distance `k` in the alphabet.<br>• **Formula:** `C = (P + k) mod 26` (P: plaintext, C: ciphertext, k: shift distance).<br>• **Characteristics:** Vulnerable to brute force because there are only 25 possible keys (`k ∈ [1, 25]`). |
| **Affine Cipher** | • **Principle:** Linear transformation combining multiplication and addition on each character.<br>• **Formula:** `C = (a · P + b) mod 26`, key pair `(a, b)`, where `b` is the shift (like Caesar) and `a` satisfies `gcd(a, 26) = 1` (so a multiplicative inverse exists for decryption).<br>• **Characteristics:** Small key space of only 312 valid keys; preserves character frequency distribution. |
| **Vigenère Cipher** | • **Principle:** Uses a repeating keyword to generate a variable shift for each position (polyalphabetic cipher).<br>• **Mechanism:** The i-th plaintext character is shifted by the corresponding keyword character: `C_i = (P_i + K_i) mod 26`.<br>• **Characteristics:** Since the shift keeps changing, identical characters at different positions produce different ciphertext characters, strongly resisting monoalphabetic frequency analysis. |
| **Hill Cipher** | • **Principle:** Groups plaintext characters into n-dimensional vectors and applies matrix multiplication to each block.<br>• **Formula:** `C = P · K mod 26` (P: plaintext vector, K: n × n key matrix).<br>• **Characteristics:** Encrypts n characters at once, effectively hiding single-letter frequency patterns. |
 
### Transposition Techniques
Transposition does **not** change the identity of characters; it only **rearranges their positions** in the plaintext. Combining substitution and transposition significantly enhances security.
 
### Weaknesses of Classical Cryptography
Classical ciphers have extremely small key spaces and fixed mapping rules, making them highly susceptible to **brute-force attacks** and **frequency analysis**.
 
---
 
## 4. Block Ciphers
 
### Overview
A block cipher splits data into fixed-size blocks (e.g., 64 or 128 bits) and encrypts them simultaneously with a secret key. Unlike classical ciphers that process characters independently, block ciphers iterate over multiple rounds with distinct subkeys, combining substitution and permutation. This diffuses and confuses the data thoroughly, creating a strong avalanche effect while remaining reversible so the receiver can recover the original plaintext.
 
Two prevalent architectures:
 
* **SPN:** Substitution-Permutation Network.
* **Feistel Network.**
### DES and Triple DES (3DES)
 
* **DES (Data Encryption Standard):** A legacy block cipher standard released in **1977**. Processes **64-bit** blocks with a **56-bit key** (plus 8 parity bits, 64 bits total) over a **16-round Feistel** structure. The pipeline:
  1. Initial Permutation (**IP**).
  2. 16 round transformations. In each round, function **F** expands the right half via expansion **E**, XORs it with a 48-bit round key, passes it through 8 non-linear **S-boxes** (confusion), then applies permutation **P**.
  3. Swap the two halves.
  4. Inverse Initial Permutation (**IP⁻¹**).
* **Triple DES (3DES):** A transitional solution once DES became insecure. Runs DES three times in an Encryption-Decryption-Encryption (**EDE**) configuration: `C = E_K3(D_K2(E_K1(M)))`. Uses up to 3 keys with a nominal length of **168 bits**, giving an effective security of about **112 bits** (due to meet-in-the-middle attacks).
#### Weaknesses of DES and 3DES
 
* **DES:** With only a 56-bit key (2⁵⁶ ≈ 7.2 × 10¹⁶ possibilities), the key space is too small for modern computing power. DES was broken by EFF's *Deep Crack* machine in under 3 days in 1998, and can be brute-forced today within hours using GPU clusters.
* **3DES:** Fixes the key length problem but runs about **3× slower** than DES. Its 64-bit block size also exposes it to collision risks over large data volumes (**Sweet32** vulnerability), so NIST has fully deprecated it.
### Advanced Encryption Standard (AES)
AES was published by NIST in **2001** to replace DES. It uses a fixed **128-bit** block size and three key-length variants:
 
| Variant | Key Length | Rounds |
| :--- | :---: | :---: |
| AES-128 | 128 bits | 10 |
| AES-192 | 192 bits | 12 |
| AES-256 | 256 bits | 14 |
 
AES uses an **SPN** structure operating on a **4 × 4 byte state matrix**, and all internal transformations are fully invertible.
 
#### AES Round Operations
 
* **SubBytes:** Substitutes each byte via a non-linear S-box to create confusion.
* **ShiftRows:** Cyclically shifts matrix rows to the left to create diffusion.
* **MixColumns:** Mixes column bytes by multiplication over the Galois Field GF(2⁸) for stronger diffusion.
* **AddRoundKey:** Bitwise XOR between the state matrix and the round subkey (derived via key expansion).
### Block Cipher Modes of Operation
 
* **ECB (Electronic Codebook):** Encrypts each block independently. Critical flaw: identical plaintext blocks yield identical ciphertext blocks, exposing data patterns (e.g., preserving image outlines). **Rule: never use ECB for real-world data.**
* **CBC (Cipher Block Chaining):** XORs each plaintext block with the previous ciphertext block before encryption; the first block needs a random **Initialization Vector (IV)**. Identical plaintext blocks at different positions produce different ciphertext blocks. *Disadvantages:* encryption cannot be parallelized and padding is required.
* **CFB (Cipher Feedback):** Turns a block cipher into a stream cipher by encrypting the previous ciphertext and XORing it with the current plaintext: `C_i = P_i ⊕ E_K(C_{i-1})`. *Feature:* self-synchronizing over noisy network channels.
* **OFB (Output Feedback):** Generates an independent keystream by feeding back the encrypted output: `C_i = P_i ⊕ O_i`. Separates keystream generation from plaintext processing.
* **CTR (Counter):** Encrypts a counter/nonce that increments per block, then XORs it with the plaintext. Highly efficient, parallelizable, and needs no padding. **Vital rule: never reuse a (Key, Nonce/IV) pair.**
* **GCM (Galois/Counter Mode):** Extends CTR with built-in data authentication (**AEAD**). Provides confidentiality, integrity and authenticity; widely used in TLS (HTTPS) and modern Wi-Fi.
---
 
## 5. Stream Ciphers
 
* **Fundamental Mechanism:** Generates a pseudorandom keystream and combines it via bitwise XOR directly with the plaintext bits/bytes to produce the ciphertext.
* **Practical Examples:**
  * **RC4:** Legacy algorithm, now deprecated and prohibited due to security flaws.
  * **ChaCha20:** Modern stream cipher with high performance and strong security.
* **Vital Security Rule:** Never reuse the same Key and Nonce combination. Key reuse exposes plaintexts through XOR subtraction: `C1 ⊕ C2 = P1 ⊕ P2`.
* **Linear Feedback Shift Registers (LFSR):** Hardware component for generating pseudorandom bits with low complexity. Because of their linearity, standalone LFSRs are vulnerable to algebraic reconstruction (e.g., the Berlekamp–Massey algorithm) and are insecure on their own.
 
