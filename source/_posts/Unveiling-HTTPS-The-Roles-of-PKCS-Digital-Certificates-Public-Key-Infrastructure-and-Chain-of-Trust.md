---
title: >-
  Unveiling HTTPS: The Roles of PKCS, Digital Certificates, Public Key
  Infrastructure, and Chain of Trust
date: 2024-06-26 22:00:11
description: "Unveiling HTTPS: The Roles of PKCS, Digital Certificates, Public Key Infrastructure, and Chain of Trust"
categories: HTTPS
tags:
- HTTPS
- PKCS
- Digital Certificates
- Public Key Infrastructure
- PKI
- Chain of Trust
---

In network communications, we often need to protect the content of our messages, and this is done by hiding the original content through encryption. HTTPS (HyperText Transfer Protocol Secure) uses a combination of symmetric and asymmetric encryption to ensure secure communication over the internet. Here's how it works:
<!--more-->

1. TCP Handshake: The initial connection between the client and server is established using TCP (Transmission Control Protocol). This handshake ensures a reliable connection.

2. SSL/TLS Handshake:
    - Client Hello: The client sends a message indicating its support for SSL/TLS version and set of necessary encryption algorithms (cipher suites) it prefers.
    - Server Hello: The server responds so let browser knows whether it can support the algorithms and TLS version. The server then sends the SSL certificate to the client. The certificate contains the public key, hostname, expiry dates, etc.
    - Certificate Verification: The client verifies the server's certificate to ensure it belongs to the intended party.
    - Key Exchange: The client uses this public key(parse from certificate) to encrypt a randomly generated session key.
    - Finished: Now that both the client and the server hold the same session key (symmetric encryption),.
3. Data Encryption:
    - Symmetric Encryption: All subsequent data is encrypted using the shared secret key. This is much faster than asymmetric encryption and is ideal for large amounts of data.

## Symmetric Encryption

Symmetric-key algorithms are algorithms for cryptography that use the same cryptographic keys for both the encryption of plaintext and the decryption of ciphertext. The key represent a shared secret between two or more parties that can be used to maintain a private information link.

- Example Algorithms: AES (Advanced Encryption Standard), DES (Data Encryption Standard). 

- For more Algorithms and their difference, reference: [Popular Symmetric Algorithms](https://cryptobook.nakov.com/symmetric-key-ciphers/popular-symmetric-algorithms), [Symmetric-key algorithm](https://en.wikipedia.org/wiki/Symmetric-key_algorithm)

This solution appears ideal, but there's a significant issue: how can multiple parties agree on the encryption key? If the key is transmitted over the network, it could be intercepted by hackers. Consequently, symmetric encryption is perpetually caught in this dilemma. This is where asymmetric encryption becomes essential.

## Asymmetric Encryption(Public key cryptography)

Asymmetric Encryption use pairs of related keys. Each key pair consists of a public key and a corresponding private key. Anyone with a public key can encrypt a message, yielding a ciphertext, but only those who know the corresponding private key can decrypt the ciphertext to obtain the original message.

- Example Algorithms: RSA (Rivest-Shamir-Adleman), ECC (Elliptic Curve Cryptography).

- For more Algorithms and their difference, reference: [Asymmetric Algorithms](https://dev.digicert.com/en/trustcore-sdk/nanocrypto/asymmetric-algorithms.html), [Public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography)

## PKCS(Public Key Cryptography Standards)

Public Key Cryptography Standards (PKCS) are protocols that structure various aspects of using public key infrastructure for information exchange. It includes standards for key exchange, password-based encryption, cryptographic messaging, private key information, certification requests, cryptographic tokens, and personal information exchange.

RSA, a fundamental cryptographic algorithm, is utilized within various PKCS standards to ensure secure and standardized implementation of public key cryptography. As one of the first public key cryptosystems, the RSA algorithm is widely used for secure data transmission and underpins many protocols within the PKCS suite.

**Some PKCS Standards Related To RSA**:

- **PKCS #1 RSA Cryptography Standard**:
    
    This is the a standard specifically for RSA cryptography. It defines the mathematical properties and formats for RSA public and private keys, as well as the encryption and signature schemes based on RSA. It includes details on padding schemes (such as OAEP and PSS) to ensure the security of RSA operations. 
    
    - **PKCS #1 v1.5**  and **PKCS #1 v2.0(OAEP)**  are the primary padding schemes for RSA encryption, RSA can be used with various padding schemes, particularly in the context of digital signatures. OAEP is generally preferred for its stronger security properties
    - Standard RSA private key decryption involves removing all padding or encoding from the ciphertext, resulting in the plaintext as it was originally encrypted. Prior to decryption, it is essential to determine the specific padding mode used during encryption. Additionally, the length of the input to any RSA decryption method must match the size of the RSA key in bytes (i.e., the modulus size in bytes).  For OAEP (Optimal Asymmetric Encryption Padding), it is also crucial to know which hash functions were used during the encoding process.
- **PKCS #7 Cryptographic Message Syntax Standard**:
    
    This standard defines a general syntax for data that may have cryptography applied to it, such as digital signatures and encryption. RSA is often used within PKCS #7 for signing and encrypting messages.
    
- **PKCS #8 Private-Key Information Syntax Standard**:
    This is a more versatile standard for storing private keys. It defines the structure for private key information and supports a variety of key types, including RSA, DSA, and ECDSA. The standard specifies how private keys should be encoded and securely stored.
    
- **PKCS #10 Certification Request Standard:**
    
    This standard is used when applying for a digital certificate from a certificate authority.
    
- **PKCS #12 Personal Information Exchange Syntax Standard**:
    
    This standard defines how to securely store and transport a user’s private keys, certificates, and other related information. It is used extensively for exporting and importing digital certificates.

More PKCS details:
- [認識 PKI 架構下的數位憑證格式與憑證格式轉換的心得分享](https://blog.miniasp.com/post/2018/04/21/PKI-Digital-Certificate-Format-Convertion-Notes)
- [wiki PKCS](https://en.wikipedia.org/wiki/PKCS)
- [What are Public Key Cryptography Standards (PKCS)? Meaning, Specifications, and Importance](https://www.spiceworks.com/it-security/network-security/articles/what-is-pkcs/)
- [IETF DataTracker PKCS #1: RSA Cryptography Specifications Version 2.2](https://datatracker.ietf.org/doc/html/rfc8017)
- [PKCS#1 and PKCS#8 format for RSA private key](https://stackoverflow.com/questions/48958304/pkcs1-and-pkcs8-format-for-rsa-private-key)


Asymmetric encryption also encounters a similar challenge: how do you share the public key with the other party? One approach is to post it on a public web address for the other party to download; another is to send it during the connection establishment. Both methods share a common problem: how do you verify that the public key you received is legitimate? Could someone impersonate the website and provide you with their own public key?

This is where Public Key Infrastructure (PKI) and Digital certificate(Public key certificates) comes in.

## Public Key Infrastructure (PKI)

A Public Key Infrastructure (PKI) is a system that binds public keys with the identities of entities such as individuals and organizations. This binding is achieved through a registration and certificate issuance process conducted by a trusted entity known as a Certificate Authority (CA). Depending on the security requirements, this process can be automated or involve human oversight.

PKI provides essential "trust services," which essentially means trusting the actions or outputs of entities, whether they are people or computers. These trust services aim to achieve one or more of the following objectives: Confidentiality, Integrity, and Authenticity (often abbreviated as CIA).

Here’s an overview of what is involved in PKI:

- **Certificate Authority (CA):** An entity that stores, issues, and signs digital certificates.
- **Registration Authority (RA):** An entity that verifies the identities of entities (individuals, organizations, applications, etc.) requesting certificates.
- **Central Directory:** A repository where keys are stored and retrieved.
- **Certificate Management System:** A system that manages the lifecycle of certificates, including issuance, access, and delivery.
- **Certificate Policy:** A set of rules that defines the procedures for managing certificates and keys within the PKI framework, ensuring their security.

For more detailed information: [Wikipedia page on Public Key Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure).

Now that you understand what a PKI (Public Key Infrastructure) is, let's delve into how you can request digital certificates and authenticate them.

## Digital certificate (Public key certificates)

### Generating a Key Pair

First, generate a pair of cryptographic keys—a public key and a private key. The public key is shared openly, while the private key is kept secure and known only to you.

1. generate private key

```bash
# NOTICE: kept private key secure
$ openssl genrsa -out yoursite_private.key 1024
$ cat yoursite_private.key

-----BEGIN PRIVATE KEY-----
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAKLp3YaO0SuRvokz
DD91tVnY0wxQI3powVlV0EcXX19t2CcCYu2glkYTKqGdpECY1iV2QQXBZiZ2dx5N
TRfVllYv4ThLhOydUjFGWL0+z1FbwkyyUYusyRbkpc7xG2oZ/UijFM8y47LqKIzK
C5sqSsrxESR4csGkljkp7KTHtT4jAgMBAAECgYA4JbfmlzQ5+uobKQ/Qk0XkaFkc
hkYj+xSgMHYu+jwxjI8Rqr3jvhPspNBtkQI6DTLJCH+Sdzw4h124gNXQIBnGn4kt
K9FadXSJRpqnulhEkPkA/NyAcKHzjm4GpWa5bA5+BrLJScBq00VK6cmhXfP3jFSJ
D4nZd+4Eck9wvaGGOQJBANiIghGCKQ3q3BUig1S3NXRHVDp69j8T+cDstNuvK1Wp
+dMjpbzWOhloRK0ieCdVa0TRXhaylPuomNpI4m1hjO8CQQDAm3MPyS8toCz0ggkH
9Y2DVqJmTI4vQ6in+9Lya1z2iSp2Tc5Lyqm/ZcFc1uBCYSdoYWX5cgEMoKRJHNPx
PEoNAkEAkc4J14RP5ME7BThCOu9LHUtSmjZmTj9DM/ewKSWhBoP4Z4ZfefK/GJCv
fe3x/np0Sti4hIwn6fWzR3lAjurbHQJAVqOWVnuBJVzv2+zCczoZtgK6epnlO42L
yESW10VERAHff+fv7Ff1k4sKN+DQcAuT1ng5jsOhhTSdseWt0M314QJBAMTFneqp
vO4QEmhHHhf4vqAiZ16IpTRl4XgfN50gpr+GoHshVSdxsk1cW3dTH2pNoiVBL56R
esjnNHpPWmkIJME=
-----END PRIVATE KEY-----
```

2. generate public key from private key

```bash
$ openssl rsa -in yoursite_private.key -pubout -out yoursite_public.pem
$ cat yoursite_public.key

-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCi6d2GjtErkb6JMww/dbVZ2NMM
UCN6aMFZVdBHF19fbdgnAmLtoJZGEyqhnaRAmNYldkEFwWYmdnceTU0X1ZZWL+E4
S4TsnVIxRli9Ps9RW8JMslGLrMkW5KXO8RtqGf1IoxTPMuOy6iiMygubKkrK8REk
eHLBpJY5Keykx7U+IwIDAQAB
-----END PUBLIC KEY-----
```

3. Then you can encrypt and decrypt a message by pair of cryptographic keys.

```bash
# PKCS#1 v2.0(RSA-OAEP)
# encrypt
openssl pkeyutl -encrypt -in plaintext.txt -pubin -inkey yoursite_public.pem -pkeyopt rsa_padding_mode:oaep -out ciphertext.bin
# decrypt
openssl pkeyutl -decrypt -in ciphertext.bin -inkey yoursite_private.key -pkeyopt rsa_padding_mode:oaep -out decrypted.txt
```

Since that every entity can generate their own key pair, how can others verify that this key pair indeed represents your identity? This is where the involvement of authoritative bodies becomes crucial. A third-party validation authority (VA) can provide the necessary entity information on behalf of the certification authority (CA).



### Generate Certificate Signing Request (CSR):

A CSR includes your public key and information about your identity. 

```bash
# Run the OpenSSL command to generate the CSR:
$ openssl req -new -key yoursite_private.key -out yoursite.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:TW
State or Province Name (full name) [Some-State]:Taipei
Locality Name (eg, city) []:Taipai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:YourSite
Organizational Unit Name (eg, section) []:Test
Common Name (e.g. server FQDN or YOUR name) []:Test
Email Address []:test

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:test
An optional company name []:test

$ cat yoursite.csr

-----BEGIN CERTIFICATE REQUEST-----
MIIB3zCCAUgCAQAwdTELMAkGA1UEBhMCVFcxDzANBgNVBAgMBlRhaXBlaTEPMA0G
A1UEBwwGVGFpcGFpMREwDwYDVQQKDAhZb3VyU2l0ZTENMAsGA1UECwwEVGVzdDEN
MAsGA1UEAwwEVGVzdDETMBEGCSqGSIb3DQEJARYEdGVzdDCBnzANBgkqhkiG9w0B
AQEFAAOBjQAwgYkCgYEAoundho7RK5G+iTMMP3W1WdjTDFAjemjBWVXQRxdfX23Y
JwJi7aCWRhMqoZ2kQJjWJXZBBcFmJnZ3Hk1NF9WWVi/hOEuE7J1SMUZYvT7PUVvC
TLJRi6zJFuSlzvEbahn9SKMUzzLjsuoojMoLmypKyvERJHhywaSWOSnspMe1PiMC
AwEAAaAqMBMGCSqGSIb3DQEJAjEGDAR0ZXN0MBMGCSqGSIb3DQEJBzEGDAR0ZXN0
MA0GCSqGSIb3DQEBCwUAA4GBACl5YxcubqBt9HQ6sA0+M40xf8EyU+/xYYxi7j7p
LqWMAupBsbicX3zsB7dP8AuT2cQkbCt6AGzuPyLA2aDSlcY1a7f4LbpMIP268LPs
zGTFGbQGM+Y2xh7uPBZyF7FJOlX//usRnzDsss8OLoCgHxNomIwvFKWvemxB2k/a
jtgC
-----END CERTIFICATE REQUEST-----

# Verify the CSR: you can inspect the contents of the CSR to ensure that the information is correct:
$ openssl req -in yoursite.csr -noout -text

Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: C=TW, ST=Taipei, L=Taipai, O=YourSite, OU=Test, CN=Test, emailAddress=test
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (1024 bit)
                Modulus:
                    00:a2:e9:dd:86:8e:d1:2b:91:be:89:33:0c:3f:75:
                    b5:59:d8:d3:0c:50:23:7a:68:c1:59:55:d0:47:17:
                    5f:5f:6d:d8:27:02:62:ed:a0:96:46:13:2a:a1:9d:
                    a4:40:98:d6:25:76:41:05:c1:66:26:76:77:1e:4d:
                    4d:17:d5:96:56:2f:e1:38:4b:84:ec:9d:52:31:46:
                    58:bd:3e:cf:51:5b:c2:4c:b2:51:8b:ac:c9:16:e4:
                    a5:ce:f1:1b:6a:19:fd:48:a3:14:cf:32:e3:b2:ea:
                    28:8c:ca:0b:9b:2a:4a:ca:f1:11:24:78:72:c1:a4:
                    96:39:29:ec:a4:c7:b5:3e:23
                Exponent: 65537 (0x10001)
        Attributes:
            unstructuredName         :test
            challengePassword        :test
            Requested Extensions:
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        29:79:63:17:2e:6e:a0:6d:f4:74:3a:b0:0d:3e:33:8d:31:7f:
        c1:32:53:ef:f1:61:8c:62:ee:3e:e9:2e:a5:8c:02:ea:41:b1:
        b8:9c:5f:7c:ec:07:b7:4f:f0:0b:93:d9:c4:24:6c:2b:7a:00:
        6c:ee:3f:22:c0:d9:a0:d2:95:c6:35:6b:b7:f8:2d:ba:4c:20:
        fd:ba:f0:b3:ec:cc:64:c5:19:b4:06:33:e6:36:c6:1e:ee:3c:
        16:72:17:b1:49:3a:55:ff:fe:eb:11:9f:30:ec:b2:cf:0e:2e:
        80:a0:1f:13:68:98:8c:2f:14:a5:af:7a:6c:41:da:4f:da:8e:
        d8:02
```

After running these steps, you'll have a `yoursite.csr` file that you can submit to a Certificate Authority (CA) to obtain an SSL/TLS certificate.

A Certificate Authority (CA) may run below command to generate Certificate.

```bash
$ openssl x509 -req -in yoursite.csr -CA cacertificate.pem -CAkey caprivate.key -out yoursite.pem
```

Reference:
- Explains the various ways in which RSA keys can be stored:  [RSA Key Formats](https://www.cryptosys.net/pki/rsakeyformats.html)
- [Difference between pem, crt, key files](https://stackoverflow.com/questions/63195304/difference-between-pem-crt-key-files)

### Self-signed certificate

For for testing or internal use, you can also generate self-signed certificate using OpenSSL, you'll need both a private key and a certificate signing request (CSR). However, you can generate a self-signed certificate directly without a CSR. Here’s how to do it:

```bash
$ openssl req -new -x509 -key yoursite_private.key -out yoursite_certificate.crt -days 365
$ openssl x509 -in yoursite_certificate.crt -text -noout

Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3d:b5:59:d5:c9:77:54:96:91:d4:9f:c3:27:f4:34:8a:58:a3:2a:3a
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=TW, ST=Taipei, L=Taipei, O=YourSite
        Validity
            Not Before: Jun 26 01:58:40 2024 GMT
            Not After : Jun 26 01:58:40 2025 GMT
        Subject: C=TW, ST=Taipei, L=Taipei, O=YourSite
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (1024 bit)
                Modulus:
                    00:a2:e9:dd:86:8e:d1:2b:91:be:89:33:0c:3f:75:
                    b5:59:d8:d3:0c:50:23:7a:68:c1:59:55:d0:47:17:
                    5f:5f:6d:d8:27:02:62:ed:a0:96:46:13:2a:a1:9d:
                    a4:40:98:d6:25:76:41:05:c1:66:26:76:77:1e:4d:
                    4d:17:d5:96:56:2f:e1:38:4b:84:ec:9d:52:31:46:
                    58:bd:3e:cf:51:5b:c2:4c:b2:51:8b:ac:c9:16:e4:
                    a5:ce:f1:1b:6a:19:fd:48:a3:14:cf:32:e3:b2:ea:
                    28:8c:ca:0b:9b:2a:4a:ca:f1:11:24:78:72:c1:a4:
                    96:39:29:ec:a4:c7:b5:3e:23
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                42:37:D1:41:A1:BC:B9:12:61:76:FE:F4:BD:80:F3:6D:96:75:21:FC
            X509v3 Authority Key Identifier:
                42:37:D1:41:A1:BC:B9:12:61:76:FE:F4:BD:80:F3:6D:96:75:21:FC
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        33:f3:3f:5f:9c:63:a0:10:1e:90:d1:50:4e:d3:e2:6f:19:24:
        98:41:57:65:32:48:a5:89:4e:b3:ea:25:95:8b:d0:ee:44:5e:
        28:b1:1c:3d:65:3e:37:95:e0:cd:6e:7b:2f:5c:8a:02:4e:99:
        df:42:8b:a7:6b:f7:23:b4:47:15:7e:93:a5:8f:91:e8:29:21:
        56:5a:f9:ab:9f:a1:24:d7:cf:5a:40:aa:57:02:30:a5:db:be:
        2d:b1:c8:53:6c:14:86:9c:2b:a4:7e:0e:84:ff:53:a7:1b:02:
        28:27:ca:4a:63:70:c0:de:84:33:66:f7:d9:aa:a0:b3:ba:80:
        f8:8a
```

the contents of this certificate:

- Issuer: This indicates who issued the certificate.
- Subject: Specifies to whom the certificate was issued.
- Validity: Indicates the duration of validity for the certificate.
- Public Key: Contains the content of the public key.
- Signature Algorithm: Specifies the algorithm used for signing.

Reference:
  - [如何使用 OpenSSL 建立開發測試用途的自簽憑證 (Self-Signed Certificate)](https://blog.miniasp.com/post/2019/02/25/Creating-Self-signed-Certificate-using-OpenSSL)


### Digital signature
A digital signature is a cryptographic mechanism used to verify the authenticity and integrity of a message, software, or digital document. It functions like a handwritten signature or a stamped seal, but it is much more secure.

When we submit a certificate request to a Certificate Authority (CA), the CA will sign the certificate. This process is known as the signature algorithm. But how can we ensure that the signature is indeed from the CA? The answer lies in using something that only the CA privately owns, which is the CA's private key.

For example, if Google needs to prove to users that it is indeed Google, it must find a reliable CA. The CA's staff will verify various documents and proofs of identity and then issue a digital certificate for Google.

Finally, this certificate contains all of Google's identity information and a digital signature signed by the CA.

- The digital signature algorithm works as follows:
  - **Signing**: A certificate contains three parts: certificate content, hash algorithm, and encrypted ciphertext. The certificate content is hashed using the hash algorithm to produce a hash value, which is then encrypted using the private key provided by the CA institution.

  - **Verification**: This certificate has an issuing authority, the CA. You just need to obtain the public key of this CA and use it to decrypt the signature part in certificate. If the decryption is successful and the hash matches, it means that the certificate is valid.

  <img src="./Digital_Signature_diagram_svg.png" width="70%" height="70%" title="Digital_Signature_diagram" alt="https://io.wikipedia.org/wiki/Bica_signato#/media/Arkivo:Digital_Signature_diagram.svg">

Reference
- [Digital Signature](https://en.wikipedia.org/wiki/Digital_signature)


## **Chain Of Trust**

The main difference between a CA-signed certificate and a self-signed certificate is that a self-signed certificate contains a single CERTIFICATE block, whereas a CA-signed certificate contains multiple CERTIFICATE blocks.

- Self-signed certificate contains a single CERTIFICATE block, encompassing all necessary information such as the public key, identity details, and digital signature. While self-signed certificates can be convenient for testing or internal use, they lack the endorsement of a trusted third party, which is essential for establishing widespread trust in the certificate's authenticity.
- On the other hand,  CA-signed certificates are issued and signed by a trusted Certificate Authority. These certificates typically include multiple CERTIFICATE blocks within a chain. The primary CERTIFICATE block represents the certificate being issued (end-entity certificate), while subsequent blocks (intermediate certificates) link the end-entity certificate to the root certificate of the CA. This chain of certificates forms the basis of the "chain of trust," where each certificate in the chain is validated against the next until the root CA certificate is reached.

```bash

# self signed contains single CERTIFICATE block
-----BEGIN CERTIFICATE-----
MIICYDCCAcmgAwIBAgIUPbVZ1cl3VJaR1J/DJ/Q0ilijKjowDQYJKoZIhvcNAQEL
BQAwQjELMAkGA1UEBhMCVFcxDzANBgNVBAgMBlRhaXBlaTEPMA0GA1UEBwwGVGFp
cGVpMREwDwYDVQQKDAhZb3VyU2l0ZTAeFw0yNDA2MjYwMTU4NDBaFw0yNTA2MjYw
MTU4NDBaMEIxCzAJBgNVBAYTAlRXMQ8wDQYDVQQIDAZUYWlwZWkxDzANBgNVBAcM
BlRhaXBlaTERMA8GA1UECgwIWW91clNpdGUwgZ8wDQYJKoZIhvcNAQEBBQADgY0A
MIGJAoGBAKLp3YaO0SuRvokzDD91tVnY0wxQI3powVlV0EcXX19t2CcCYu2glkYT
KqGdpECY1iV2QQXBZiZ2dx5NTRfVllYv4ThLhOydUjFGWL0+z1FbwkyyUYusyRbk
pc7xG2oZ/UijFM8y47LqKIzKC5sqSsrxESR4csGkljkp7KTHtT4jAgMBAAGjUzBR
MB0GA1UdDgQWBBRCN9FBoby5EmF2/vS9gPNtlnUh/DAfBgNVHSMEGDAWgBRCN9FB
oby5EmF2/vS9gPNtlnUh/DAPBgNVHRMBAf8EBTADAQH/MA0GCSqGSIb3DQEBCwUA
A4GBADPzP1+cY6AQHpDRUE7T4m8ZJJhBV2UySKWJTrPqJZWL0O5EXiixHD1lPjeV
4M1uey9cigJOmd9Ci6dr9yO0RxV+k6WPkegpIVZa+aufoSTXz1pAqlcCMKXbvi2x
yFNsFIacK6R+DoT/U6cbAignykpjcMDehDNm99mqoLO6gPiK
-----END CERTIFICATE-----

# CA signed contains multiple CERTIFICATE block

-----BEGIN CERTIFICATE-----
[End-entity Certificate]
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
[Intermediate Certificate 1]
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
[Intermediate Certificate 2]
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
[Root Certificate]
-----END CERTIFICATE-----
```

The chain of trust is critical in ensuring the integrity and security of digital communications. By relying on CA-signed certificates and their associated chains, users can verify the legitimacy of a certificate without directly verifying the identity of the entity themselves. This process helps prevent various security threats, including man-in-the-middle attacks and unauthorized access.
