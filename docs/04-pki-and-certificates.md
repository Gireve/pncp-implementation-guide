## 4 APPENDIX 1: About PKI and digital certificates

### 4.1.1 What is a digital certificate?

A digital certificate is a digital document used to verify the identity of a device or entity and establish secure communication over a network.

It contains the signature of a certification authority (CA), which attests to the authenticity of the information and the reliability of the certificate owner.

It also contains other information such as the subject's distinguished name or DN (element 1 of the displayed certificate), a public key (element 2 of the displayed certificate), certificate validity period (element 3 of the displayed certificate), and issuer's name (element 4 of the displayed certificate. It is the CA's DN which signed the certificate).

![Certificate example](./images/media/image14.png)

The DN identifies the subject, typically a device or entity, and includes information such as the name, location, and domain name.

The validity period determines how long the certificate can be used for secure communication. For a Root-CA it can be 20 years or more, for leaf certificates it is much shorter, typically a couple of months.

### 4.1.2 How do we make them?

To obtain a digital certificate, an entity must first create a pair of keys. The entity must then create with its key pair a Certificate Signing Request (CSR) containing their public key and DN and submits it to a trusted Certificate Authority (CA) to have the certificate issued.

The Registration Authority (RA) of that CA validates the subject's identity, the CA verifies then the information in the CSR (validity of the signature, algorithm used, etc…), and issues a signed digital certificate.

When it issues the signed digital certificate, the CA adds several pieces of information to the certificate in addition to the information provided in the Certificate Signing Request (CSR). This includes:

- **Issuer Name:** The CA's name is added to the certificate as the issuer.
- **Serial Number:** The certificate is given a unique serial number to identify it.
- **Validity Period:** The CA sets the validity period for the certificate, which specifies the period for which the certificate will be considered valid.
- **Digital Signature:** The CA digitally signs the certificate to ensure its authenticity and integrity.
- **Extensions:** The CA may add extensions to the certificate that provide additional information or functionality, such as indicating the intended usage of the certificate or providing a link to the CA's certificate revocation list (CRL).
- **Subject DN:** While the DN in the CSR is typically used as the DN in the issued certificate, the CA may add or modify some information in the DN for consistency with their own naming conventions.

### 4.1.3 The concept of certificate chain

A digital certificate chain is a hierarchical structure used in Public Key Infrastructure (PKI) to establish the authenticity and trustworthiness of digital identities. At its core is the Root Certificate Authority (Root CA), a trusted entity that issues certificates. These certificates include leaf certificates (end-entity certificates) which belong to specific users, devices, or services.

PKI involves a network of CAs that issue and verify these certificates. The Root CA, being the highest authority, signs its own certificate, forming the foundation of trust. Intermediate CAs, in turn, sign certificates of lower-level CAs or end entities. This chain of trust connects the Root CA to the leaf certificates.

![Certificate chain diagram](./images/media/image15.png)

### 4.1.4 Explanation about certificate chain validation

To validate a leaf certificate, we need to validate the whole certificate chain, which means verifying that all intermediate certificates between the leaf certificate and the trusted root certificate are valid and trustworthy.

This is necessary because a leaf certificate is only as trustworthy as the chain of certificates leading back to the trusted root certificate. Each intermediate certificate in the chain is responsible for verifying the authenticity of the certificate below it in the chain.

The process of validating the certificate chain involves checking the signature of each certificate, verifying the certificate's expiration date, and ensuring that the certificate was issued by a trusted authority. If any certificate in the chain fails to pass these checks, the entire chain is considered invalid, and the leaf certificate is not trusted.

![Certificate chain validation examples](./images/media/image16.png)

*Example with a verified Certificate Chain — Example with a certificate chain that cannot be verified.*

*from [https://docs.oracle.com/cd/E19424-01/820-4811/gdzea/index.html](https://docs.oracle.com/cd/E19424-01/820-4811/gdzea/index.html)*

> Validation of the certificate chain is done with a trusted root certificate previously installed in the system because it is the ultimate authority in the chain. The trusted root certificate is the foundation of the chain, and all other certificates in the chain ultimately derive their trustworthiness from it. If the root certificate is not trusted, then the entire chain is not trusted, and the leaf certificate is considered invalid.
> All trusted root certificates must be communicated in a secure way by a trustworthy party and must be installed in a secure way in the system (in a Truststore or another secure location).

### 4.1.5 About certificate revocation

Certificate revocation is the process of invalidating a previously issued digital certificate before its expiration date. This is necessary when a certificate's private key is compromised, the certificate holder's status changes, or other security concerns arise. Revocation prevents unauthorized parties from using compromised or outdated certificates to impersonate legitimate entities.

**Certificate Revocation Lists (CRLs)** are time-stamped lists published by Certificate Authorities (CAs) that contain information about revoked certificates. These lists help systems and applications verify whether a certificate is still valid or has been revoked.

**An Online Certificate Status Protocol (OCSP)** responder is an alternative to CRLs. It provides real-time certificate status information by enabling clients to query a CA or OCSP responder server about the current validity of a certificate. This reduces the need to download and parse lengthy CRLs, offering more timely and efficient certificate validation.

During certificate chain validation, the CRL or OCSP responder information of each CA in the chain is specified in the certificates' extensions. The validation process involves checking if any certificate in the chain has been revoked by consulting the CRL or OCSP responder. If a certificate is found to be revoked, it will be rejected, enhancing the overall security of the PKI system, and ensuring the trustworthiness of digital identities.

### 4.1.6 About certificate renewal

Digital certificate renewal involves extending the validity period of an existing certificate. This process is important to maintain secure communication and identity verification without the need to create an entirely new certificate from scratch. Renewal can occur in two main ways: by reusing the same key pair or by generating a new key pair (rekey).

**Renewal with Same Key Pair:** In this approach, the existing key pair (public and private key) is retained, and a new certificate is issued with an extended validity period. The certificate authority (CA) verifies the identity of the certificate holder and updates the certificate's data, such as expiration date. This process is relatively straightforward and maintains the continuity of the cryptographic keys, eliminating the need to distribute new public keys.

**Rekeying (Generating a New Key Pair):** Rekeying involves creating a new key pair (public and private key) for the certificate holder. This is often recommended for security reasons, as it helps mitigate the risks associated with long-term key exposure. Rekeying provides a fresh set of cryptographic keys, which can enhance security against potential key compromise or advances in cryptographic attacks. The CA issues a new certificate with the updated information, and the old certificate is revoked to prevent its use.

Both approaches have their advantages and trade-offs. Renewal with the same key pair is convenient and avoids the complexities of key distribution. Rekeying, on the other hand, enhances security by periodically changing the keys but requires careful key management and distribution to ensure a smooth transition without disrupting services.

Ultimately, the decision between renewal with the same key pair or rekeying depends on the balance between security requirements, operational considerations, and the specific policies of the organization managing the certificates.