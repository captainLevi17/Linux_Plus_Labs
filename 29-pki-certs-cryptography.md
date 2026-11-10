# Lab 29 - Public Key Infrastructure, Cryptography, and Digital Certificates

## Objective

Master foundational principles of applied cryptography, asymmetric vs. symmetric encryption, cryptographic hashing, digital signatures, Public Key Infrastructure (PKI), and TLS/SSL certificate mechanics on Linux.

By completing this lab, I should be able to:

* Understand asymmetric key management (public vs. private keys) and public key encryption flows

* Utilize cryptographic hashes to verify data integrity and understand digital signature validation

* Explain how TLS handshakes securely exchange symmetric session keys using asymmetric cryptography

* Define the role of Certificate Authorities (CAs), Wildcard Certificates (`*.domain.com`), and Self-Signed Certificates

* Inspect certificate files, verify SSL/TLS connections, and generate self-signed certificates on Linux using `openssl`

## Part 1 - Asymmetric Encryption & Key Management

Public Key Cryptography (Asymmetric Encryption) relies on mathematically linked key pairs: a public key and a private key.

### Key Pair Roles

* **Public Key**

  * **Distribution:** Shared freely with anyone or published on key servers.

  * **Function:** Used by senders to **encrypt** data intended for the key owner, or by receivers to **verify digital signatures** created by the key owner.

* **Private Key**

  * **Distribution:** Kept strictly secret and accessible only to the owner.

  * **Function:** Used by the owner to **decrypt** data encrypted with their matching public key, or to **create digital signatures**.

### Confidentiality Encryption Flow

To send confidential data to a recipient:
1. Senders encrypt the plaintext message using the **recipient's public key**.
2. The recipient decrypts the ciphertext using their matching **private key**.

## Part 2 - Cryptographic Hashes & Digital Signatures

### Cryptographic Hash Functions

* **Purpose:** Generate a unique, fixed-length signature (digest) from arbitrary input data to ensure **data integrity** (verifying data has not been modified, corrupted, or tampered with).

* **Common Algorithms:** SHA-256 (`sha256sum`), SHA-512 (`sha512sum`), MD5 (`md5sum`).

### Digital Signature Validation Process

Digital signatures guarantee both **authenticity** (proof of sender identity) and **integrity** (proof of non-tampering).

* **Creation (Sender):**

  1. Senders pass the file/message through a hash function to generate a digest.

  2. Senders encrypt the digest using their **private key** to form the digital signature.

* **Verification (Recipient):**

  1. Recipients decrypt the signature using the **sender's public key** to retrieve the original hash digest.

  2. Recipients compute a new hash digest directly from the received file/message.

  3. Senders and recipients compare the two hashes. If both hashes match, the signature is valid.

## Part 3 - TLS Handshake & Symmetric Session Keys

While asymmetric encryption is secure, it is computationally expensive. Secure web communication (HTTPS / TLS) uses asymmetric encryption to establish a fast **symmetric session key**.

### TLS Session Key Exchange Mechanics

1. **Client Initiation:** The client browser connects to a web server over TLS/SSL.

2. **Server Identity:** The web server presents its public key certificate.

3. **Session Key Generation:** The client generates a random **symmetric session key**.

4. **Key Encryption:** The client encrypts this symmetric session key using the **server's public key** and sends it to the server.

5. **Key Decryption:** The server decrypts the payload using its **private key** to extract the symmetric session key.

6. **Encrypted Communication:** Both client and server use the shared **symmetric session key** to encrypt and decrypt all subsequent traffic.

## Part 4 - Public Key Infrastructure & Certificates

### Certificate Authority (CA)

* A trusted third-party organization (e.g., Let's Encrypt, DigiCert) that verifies the identity of entity owners, issues public key certificates, and digitally signs certificates.

### Wildcard Certificates

* Public key certificates that cover a base domain and all of its direct first-level subdomains.

* **Domain Pattern:** `*.example.com`

* **Coverage:** Valid for `app.example.com`, `mail.example.com`, and `www.example.com` (does not cover deeper nested subdomains like `dev.app.example.com`).

### Self-Signed Certificates

* **Encryption:** Sessions using self-signed certificates are **fully encrypted** using symmetric session keys.

* **Identity & Trust:** Web browsers display security warnings because the server identity is signed by itself rather than validated by a trusted Certificate Authority (CA).

## Part 5 - Hands-On Practice Lab

### Step 1: Hashing & Integrity Verification

1. Create a test file and generate its SHA-256 hash digest:

   ```bash
   echo "Critical system payload" > /tmp/payload.txt
   sha256sum /tmp/payload.txt > /tmp/payload.txt.sha256
   ```

2. Inspect the hash output:

   ```bash
   cat /tmp/payload.txt.sha256
   ```

3. Verify the file integrity against the checksum:

   ```bash
   sha256sum -c /tmp/payload.txt.sha256
   ```

### Step 2: Generate an RSA Key Pair and Self-Signed Certificate with OpenSSL

1. Generate a 2048-bit RSA private key:

   ```bash
   openssl genrsa -out /tmp/server.key 2048
   ```

2. Generate a self-signed X.509 certificate valid for 365 days:

   ```bash
   openssl req -new -x509 -key /tmp/server.key -out /tmp/server.crt -days 365 -subj "/CN=localhost"
   ```

3. Inspect the certificate details (issuer, validity, public key algorithm):

   ```bash
   openssl x509 -in /tmp/server.crt -text -noout
   ```

### Step 3: Test and Inspect SSL/TLS Connections

1. Test an active HTTPS TLS handshake and display server certificate details:

   ```bash
   openssl s_client -connect example.com:443 -showcerts
   ```

## Completion Checklist

- [x] I understand that public keys encrypt or verify signatures, while private keys decrypt or sign

- [x] I know how digital signatures combine cryptographic hashes and sender private keys to prove integrity and identit

- [x] I can explain how TLS handshakes use asymmetric keys to exchange symmetric session keys securely

- [x] I know that Certificate Authorities (CAs) validate entity identities and issue signed certificates

- [x] I understand that Wildcard Certificates (`*.domain.com`) cover all first-level subdomains

- [x] I know that Self-Signed Certificates fully encrypt network traffic despite browser trust warnings

- [x] I can generate hashes (`sha256sum`) and inspect certificates using `openssl`
