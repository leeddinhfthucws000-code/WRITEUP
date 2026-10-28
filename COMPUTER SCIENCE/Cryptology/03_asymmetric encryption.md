# ASYMMMETRIC ENCRYPTION OVERVIEW
 
---
 
## 1. Concept & Problem-Solving
 
* **Asymmetric Encryption Principle:** Public-key cryptography operates on asymmetric encryption principles, employing mathematically linked yet distinct keys for encryption and decryption.
* **Resolution of the Key Exchange Problem:** It eliminates the prerequisite of establishing secure out-of-band key exchange channels beforehand. Users can openly publish their public keys ('open locks') on a network, enabling any sender to encrypt confidential messages without prior direct interaction.
* **Scalable Key Management:** While legacy symmetric encryption requires communicating parties to share and maintain `N(N-1)/2` individual keys, asymmetric cryptography requires each entity to manage only a single key pair, offering exceptional scalability and security for large-scale modern networks such as the Internet.
---
 
## 2. Notable Public-Key Cryptosystems & Mathematical Foundations
 
### RSA (Rivest–Shamir–Adleman)
 
Recognized as one of the earliest and most foundational public-key cryptosystems in information security. RSA employs an asymmetric key pair comprising a **Public Key** for encryption and a **Private Key** for decryption.
 
#### Detailed Operational Workflow
 
* **Key Generation:**
  * Select two distinct large prime numbers, `p` and `q`.
  * Compute modulus `n = p × q`, serving as shared key length.
  * Compute Euler's totient function `φ(n) = (p - 1)(q - 1)`.
  * Select public exponent `e` and compute private exponent `d` such that: `d × e ≡ 1 (mod φ(n))`
* **Encryption:**
  * Transform message `M` into an integer where `0 ≤ M < n`.
  * Compute ciphertext `C` using public key `(e, n)`: `C = M^e mod n`
* **Decryption:**
  * Recover original plaintext `M` using private exponent `d`: `M = C^d mod n`

| Phase | Formula |
| :--- | :--- |
| **Key Generation** | `n = p × q`, `φ(n) = (p - 1)(q - 1)`, `d × e ≡ 1 (mod φ(n))` |
| **Encryption** | `C = M^e mod n` |
| **Decryption** | `M = C^d mod n` |
 
* **Security Foundation:** Security relies on the computational complexity of the **integer factorization problem**. Thanks to this solid mathematical foundation, RSA safeguards trillions of financial transactions, communications, and sensitive data globally every day.
### ElGamal Cryptosystem
 
Based on the computational hardness of the **discrete logarithm problem** over finite fields `Zp*` (sharing mathematical roots with the Diffie–Hellman key exchange protocol). Given a large prime `p`, a generator `g`, and `y = g^a mod p`, deriving the secret exponent `a` is computationally infeasible for modern computers.
 
### Rabin Cryptosystem
 
Relies on the hardness of the **integer factorization problem** (`n = p · q`), offering a security level equivalent to RSA. Formally, breaking the Rabin cryptosystem is theoretically as difficult as factoring the modulus `n`.
 
### Elliptic Curve Cryptography (ECC)
 
Provides equivalent security levels to legacy systems like RSA while utilizing substantially smaller key sizes (e.g., a 256-bit ECC key matches the security strength of a 3072-bit RSA key). Shorter keys accelerate arithmetic computations and decrease energy, memory, and bandwidth consumption. Due to its lightweight efficiency, ECC is widely deployed in resource-constrained environments such as mobile devices, smart cards, and network security protocols like TLS.
 
---
 
## 3. Comparative Analysis: Public-Key vs. Secret-Key Cryptography
 
| Criteria | Symmetric Cryptography (Secret Key) | Public-Key Cryptography |
| :--- | :--- | :--- |
| **Number of Keys** | 1 shared key (or 2 keys easily derived from one another). | 2 distinct keys: Public Key and Private Key. |
| **Key Relationship** | Computationally trivial to derive one key from the other. | Computationally infeasible to derive the Private Key (DK) from the Public Key (EK). |
| **Key Distribution** | Requires a dedicated, secure channel established prior to transmission. | Requires only public disclosure of the Public Key. |
| **Performance / Speed** | Extremely fast; well-suited for bulk data processing. | Significantly slower computational overhead. |
 
### Practical Deployment — Hybrid Cryptosystem
 
* In real-world implementations, **public-key cryptography** is utilized to securely negotiate and exchange session keys.
* **High-speed symmetric ciphers** (such as AES) handle the actual bulk data encryption.
* This combines the security advantages of both paradigms.
