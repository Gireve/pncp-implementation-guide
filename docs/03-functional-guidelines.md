# 3. Functional Implementation Guidelines

## 3.1 PNCP Services by Roles and Actors

The list of the PNCP services by role and actors is available in our Stoplight documentation:

[https://gireve-apis.stoplight.io/docs/pncp](https://gireve-apis.stoplight.io/docs/pncp)

---

## 3.2 PNCP Services Permissions in Gireve Trust Platform

Each service is bound to a specific permission. A permission enables the access to a service. During the technical connection process, a Gireve administrator bounds service's permissions to the operator based on its subscription. New permissions can be added or removed later if needed.

> **📝 Notes:**
> If a partner makes a call for an operator on a service for which the operator does not have permission, the call will be rejected.

## 3.3 PKI Services Guidelines

### 3.3.1 Certificates Profiles: Definition and Usage

#### Definition of the "Certificate Profile"

Our system is designed to enroll digital certificates using a "Certificate Profile" (also called PKI Profile).

The Certificate Profile defines the various characteristics of the issued certificate, such as:

- The issuer (it is the SubCA2 linked to the certificate profile)
- The certificate validity period
- The DN of the issued certificate
- Other parameters (certificate extensions, algorithms used, etc.)

When enrolling for a digital certificate, the system requires both the Certificate Signing Request (CSR) and the selected Certificate Profile. This is a way to reduce complexity, by communizing recurrent parameters.

> **📝 Notes:**
> - A certificate profile is linked to a specific SubCA2. Each SubCA2 can have multiple certificate profiles associated with different characteristics.
> - Each partner, user of the certificate generation services (certificates enrolment) will be granted with at least one certificate profile describing the common characteristics of the certificates it will request. These certificate profiles will be defined during the setup phase (connection).

---

#### Determination of the Certificate DN by the Certificate Profile

**The certificate profile specifies how the DN of the issued certificate is constructed.**

Currently, our certificate profiles take the Common Name (CN) from a CSR and apply it to the DN of the issued certificate. The (CN) in the leaf certificate of the "OEM", "CPO" and "MO" (eMSP) ISO-15118 branches correspond respectively to the PCID, the CPID and the eMAId.

The other values of the certificate DN properties, such as Domain Component (DC), Country (C), Organization (O), and Organizational Unit (OU), are set from constants defined in the configuration of the certificate profile. This ensures the conformity of the certificate issued.

The (DC) value is constrained by the ISO-15118 norm. It must equal one of the following values depending on the branch: `"OEM"`, `"MO"`, `"CPS"`, `"CPO"`.

![ISO-15118 certificate DN structure diagram](./images/media/image3.png)

---

#### Affiliation of Certificate Profile to the Trust Users

Certificate profiles are used for all certificate enrolment operations in our PKI solution. The related Trust services are the following:

- [Enroll a certificate (PNCP)](https://gireve-apis.stoplight.io/docs/pncp/branches/main/afc0baf98a6e1-enroll-a-certificate-pncp)
- [Enroll a certificate (EST protocol)](https://gireve-apis.stoplight.io/docs/pncp/branches/main/7dd53d389f659-enroll-a-certificate-est-protocol)
- [Create CC, build & sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp)
- [Create a CC and build CCB](https://gireve-apis.stoplight.io/docs/pncp/wstbyvxtxtnuy-create-a-cc-and-build-ccb)

As explained above, the certificate profiles attached to an operator will deal with the following points:

- Under which SubCA2 can the operator enrol its certificates?
- What validity period will be applied to the issued certificates?
- What values will be specified in the DN of the issued certificates?

These three questions must be discussed and answered between Gireve and the operator during the technical connection, and validated before the go-live. This will determine which Certificate Profiles to assign to the operator — they may be existing Certificate Profiles or new ones created specifically to meet the operator's needs.

Gireve associates Certificate Profiles to an operator during the operator technical connection, and later if a new certificate profile needs to be added.

> **📝 Note:** If an operator tries to use a certificate profile that is not in the list associated to it by Gireve, the system will reject the request.

### 3.3.2 Leaf Certificate Enrolment and Obtention of Linked SubCAs

In the ISO-15118 standard, communications are secured through the use and validation of digital leaf certificates.

#### How does leaf certificate enrolment work?

To obtain a leaf digital certificate, an entity must:

1. Generate an asymmetric key pair.
2. Generate a Certificate Signing Request (CSR) from this key pair. A CSR contains the public key and a subject DN related to the entity. It is signed with the private key to prove that the private key is owned by the entity.
3. Submit the CSR to the appropriate Certificate Authority (CA). The CA will validate the requesting entity identity and the conformity of the CSR. It will then use its private CA private key to issue the digital certificate and send it back to the requesting entity.

ISO-15118-2 standard requires that key pairs be based on Elliptic Curve Digital Signature Algorithm (ECDSA), specifically on the **secp256r1** elliptic curve (also called prime256v1).

Gireve will reject any CSRs based on key pairs that don't respect this constraint.

#### Why certificate requesters also need the linked SubCAs?

In order to validate and authenticate a Leaf Certificate that is associated with an end entity, the verifying entity must dispose of the Certificate chain (Leaf Certificate and intermediate SubCAs). This chain must be valid and linked to a Root Certificate trusted by the verifying entity. In most cases, the verifying entity only has a Trust Store containing a list of trusted Root certificates and does not know the intermediate certificates. Therefore, to be authenticated and trusted, an entity must not only transmit its leaf certificate but also the associated intermediate SubCAs, this is why:

- The **CPO** must install the SubCAs associated with the SECC Leaf Certificate in the SECC so the SECC can pass the SECC certificate chain to the EV. This will allow the EV to authenticate the SECC.
- The **OEM** must install the SubCAs associated with the Contract Certificate in the EV so the vehicle can pass the Contract Certificate chain to the SECC. This will allow the SECC to authenticate the Contract Certificate.
- The **eMSP** must include the associated SubCAs in the Contract Certificate Bundle to allow the CPS to validate the Contract Certificate before applying the signature and so that they can be installed in the EV (see point above).
- The **CPS** must include the associated SubCAs along with the CPS leaf certificate in the Signed Bundle to allow the EV to verify the CPS chain during the certificate installation process.
- The **OEM** must include the associated SubCAs along with the Provisioning Certificate when making it available in a PCP to allow the PCP to validate the PC before integrating it and later for the eMSP to validate the PC again before using it in a Contract Certificate Bundle.

![Certificate chain diagram](./images/media/image5.png)

To obtain a leaf certificate and the linked SubCAs, Gireve recommends the use of the [Enroll a certificate (PNCP)](https://gireve-apis.stoplight.io/docs/pncp/afc0baf98a6e1-enroll-a-certificate-pncp) webservice that presents all this information in its response.

Gireve also supports [Enrollment over Secure Transport (EST) protocol](https://datatracker.ietf.org/doc/html/rfc7030). EST is widely recognized as a standard and secure protocol used for securely enrolling devices and clients into a PKI system.

Using this protocol, enrolling a leaf certificate and fetching the associated SubCAs (and the RootCA in this case) is possible through two separate services:

- [Enroll a certificate (EST protocol)](https://gireve-apis.stoplight.io/docs/pncp/7dd53d389f659-enroll-a-certificate-est-protocol) — to enroll a leaf certificate
- [Get CA certificates (EST protocol)](https://gireve-apis.stoplight.io/docs/pncp/d6a776aea79c8-get-ca-certificates-est-protocol) — to fetch the associated SubCAs

Certificates issued by these enrollment services (PNCP or EST version) will not be added automatically in Gireve certificate pools (CCP or PCP).

For the eMSP, Gireve recommends the use of eMSP dedicated services described in [Section 3.7.1](#371-creation-of-a-contract-certificate-and-making-it-available-in-ccp-for-installation-in-ev) that will also cover the key pair generation and the contract certificate bundle generation.

> **📝 Note:** Both Enroll (EST or PNCP) services take as parameters a CSR in PKCS#10 (PEM) format **without headers and line breaks**, and a `certificateProfileId` that lets you choose which CA to sign the certificate on, and for what validity period (see [Section 3.3.1 — Definition of the certificate profile](#definition-of-the-certificate-profile)).

**Example — PEM format without headers and line breaks (correct format):**

```text
MIIBXjCCAQSgAwIBAgIEZMe+lTAKBggqhkjOPQQDAjAjMQswCQYDVQQGEwJGUjEUMBIGA1UEAwwLU1VCQ0ExLUdGUjMwHhcNMjMwNzMxMTQwMDUzWhcNMjQwNzMwMTQwMDUzWjAjMQswCQYDVQQGEwJGUjEUMBIGA1UEAwwLU1VCQ0EyLUdGUjMwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAATzGQwTJpdJu8rZDuSHKUqXhr1DL9ZmxUdwYD+ec7ydYA/JHujOFE3Pdb4ZhjdgQxaY3nAeVfLMqvTN6WguD7iBoyYwJDASBgNVHRMBAf8ECDAGAQH/AgEBMA4GA1UdDwEB/wQEAwIBBjAKBggqhkjOPQQDAgNIADBFAiAVJySp/h3mDgNl1KqUo+r0vtCmaQEX9T6glLgi2emIrAIhAK78hZAYSeC63vxx+HG5Wl7GxIRF72CMhbiNkZ7TqK1e
```

**Example — PEM format with headers and line breaks (incorrect format):**

```text
-----BEGIN CERTIFICATE-----
MIIBXjCCAQSgAwIBAgIEZMe+lTAKBggqhkjOPQQDAjAjMQswCQYDVQQGEwJGUjEU
MBIGA1UEAwwLU1VCQ0ExLUdGUjMwHhcNMjMwNzMxMTQwMDUzWhcNMjQwNzMwMTQw
MDUzWjAjMQswCQYDVQQGEwJGUjEUMBIGA1UEAwwLU1VCQ0EyLUdGUjMwWTATBgcq
hkjOPQIBBggqhkjOPQMBBwNCAATzGQwTJpdJu8rZDuSHKUqXhr1DL9ZmxUdwYD+e
c7ydYA/JHujOFE3Pdb4ZhjdgQxaY3nAeVfLMqvTN6WguD7iBoyYwJDASBgNVHRMB
Af8ECDAGAQH/AgEBMA4GA1UdDwEB/wQEAwIBBjAKBggqhkjOPQQDAgNIADBFAiAV
JySp/h3mDgNl1KqUo+r0vtCmaQEX9T6glLgi2emIrAIhAK78hZAYSeC63vxx+HG5
Wl7GxIRF72CMhbiNkZ7TqK1e
-----END CERTIFICATE-----
```

> **📝 Note 2:** The API [Get CA certificates (EST protocol)](https://gireve-apis.stoplight.io/docs/pncp/d6a776aea79c8-get-ca-certificates-est-protocol) contains the SubCA2, SubCA1 and Root CA certificates linked to a certificate profile. To enroll a leaf certificate and obtain the associated certificate chain, the same `certificateProfileId` needs to be specified as a parameter in both services.

> **📝 Note 3:** Both EST service responses are in PKCS#7 format. PKCS#7 is a cryptographic data container format that can store certificates (in X.509 format), and is sometimes used to store several certificates at once (like for the *Get CA certificates* service). A PKCS#7 file has the extension `.p7b`. This format is natively recognized by Microsoft Windows and a `.p7b` file can be opened directly for consultation by clicking on it.

> **📝 Note 4:** PNCP allows OEM, CPO and eMSP to include an optional metadata object in the request body when creating or renewing a leaf certificate. This metadata enables the association of simple business classification data with the created certificate record in Gireve systems. It is not embedded in the certificate itself. This metadata can then be used to simplify reporting, filtering, and internal monitoring. For more information, see [Section 3.3.5 — Certificate Consultation Services](#335-certificate-consultation-services).

### 3.3.3 Renewal of a Leaf Certificate

The renewal of a leaf certificate is performed by the same services used for the certificate first enrolment.

**Gireve prohibits the reuse of a public key for certificate renewal** for security reasons. If the CSR entered in the request is from a key pair already used, the system will detect it and the request will be rejected. The caller must generate a new key pair and issue a CSR from it to be able to perform a certificate renewal — this is called **certificate rekeying**.

> See [Appendix 1 — About certificate renewal](./05-appendix-pki.md#416-about-certificate-renewal) for more details on the concept.

![Leaf certificate renewal diagram](./images/media/image6.png)

> **📝 Note:** A certificate can be rekeyed before expiration. In this case, the two certificates issued from the two key pairs will both be valid until the old certificate reaches its expiration date.

> **📝 Note 2:** As explained in [Section 3.3.2 — Leaf Certificate Enrolment](#332-leaf-certificate-enrolment-and-obtention-of-linked-subcas), PNCP allows metadata to also be provided when renewing any certificate.3.3.5

### 3.3.4 Revocation of a Leaf Certificate

Leaf certificates issued from [Enroll a certificate (PNCP)](https://gireve-apis.stoplight.io/docs/pncp/afc0baf98a6e1-enroll-a-certificate-pncp) or [Enroll a certificate (EST protocol)](https://gireve-apis.stoplight.io/docs/pncp/7dd53d389f659-enroll-a-certificate-est-protocol) services can be revoked with the [Revoke a certificate](https://gireve-apis.stoplight.io/docs/pncp/rs5onohqhyyi2-revoke-a-certificate) service.

Identification of the certificate to revoke is done by the couple **serial number** (in decimal value) and **issuer DN**, which is a standard way of identifying an X.509 certificate.

As a certificate can only be revoked by the certification authority that issued it:

- Only Gireve can revoke the certificates Gireve issued.
- Gireve can only revoke the certificates Gireve issued.

In Gireve Trust solution, only the operator which owns the certificate can request its revocation.

> **📝 Note:** After a revocation, Gireve updates its CRL and adds the new revoked certificate. This update is done every day. Therefore, if you revoke a certificate and immediately check the revocation status with an OCSP call or by downloading the CRL, the certificate will still be displayed as valid until the CRL is updated.

> **📝 Note 2:** Contract certificates that have not been issued from [Enroll a certificate (PNCP)](https://gireve-apis.stoplight.io/docs/pncp/afc0baf98a6e1-enroll-a-certificate-pncp) or [Enroll a certificate (EST protocol)](https://gireve-apis.stoplight.io/docs/pncp/7dd53d389f659-enroll-a-certificate-est-protocol) need to be revoked using specific services for eMSP — see [Section 3.7.4 — Contract certificate revocation and removal from CCP](#374-contract-certificate-revocation-and-removal-from-ccp).

> **📝 Note 3:** The revocation of a Contract Certificate or a Provisioning Certificate using the [Revoke a certificate](https://gireve-apis.stoplight.io/docs/pncp/rs5onohqhyyi2-revoke-a-certificate) service will revoke these certificates but will **not** automatically remove them from Gireve pools. To do so, see [Section 3.7.4](#374-contract-certificate-revocation-and-removal-from-ccp) for CC and [Section 3.8.2](#382-pc-certificate-lifecycle-management) for PC.

---

### 3.3.5 Certificate Consultation Services

Gireve PNCP includes certificate consultation services that complement enrollment, renewal and revocation operations. Their purpose is to give operators autonomous access to the list of certificates they have enrolled, without requiring manual extraction by support teams.

Two endpoints are available for the same business purpose:

- [Retrieve enrolled Certificates](https://gireve-apis.stoplight.io/docs/pncp) — returns a paginated JSON response
- [Export enrolled Certificates CSV](https://gireve-apis.stoplight.io/docs/pncp) — returns a CSV file

> **📝 Note:** Operators can only retrieve the certificates that were issued for them, regardless of their certificate status (valid, expired, revoked). Filters and parameters are detailed in the Stoplight pages of these services.

> **📝 Note 2:** The services support filtering by metadata in addition to existing filters, and metadata associated with a certificate is returned in the response. Metadata can be defined at certificate creation or renewal by using PNCP PKI services as described in [Section 3.3.2](#332-leaf-certificate-enrolment-and-obtention-of-linked-subcas) and [Section 3.3.3](#333-renewal-of-a-leaf-certificate), and also by using specific eMSP PNCP services as described in [Section 3.7.1](#371-creation-of-a-contract-certificate-and-making-it-available-in-ccp-for-installation-in-ev).

## 3.4 RCP Services Guidelines

There are 3 types of root certificates defined in the ISO-15118-2 standard:

- **V2G Root CA** — from which the CPO and CPS branches are derived, and possibly the MO or OEM branches.
- **MO Root CA** — from which the MO (eMSP) branches are derived, if not derived from a V2G Root CA.
- **OEM Root CA** — from which the OEM branches are derived, if not derived from a V2G Root CA.

All actors that generate ISO-15118 leaf certificates are required to make sure that the Root Certificates from which their leaf certificates are derived are accessible within a Root Certificate Pool (RCP). This provision enables the actors manipulating their leaf certificates to validate them by checking the certificate chains after retrieving the associated Root Certificates from the RCP.

---

### 3.4.1 Making a Root Certificate Available in the Gireve RCP

All eMSPs and OEMs connected to Gireve's Trust platform that do not use Gireve's PKI service to generate their leaf certificates are required to make their root certificates available to Gireve for inclusion in its RCP.

**For security reasons, the provision of a root certificate in Gireve RCP is not an automated process.** The Root CA must be communicated by the operator to a Gireve Trust administrator using Gireve's secured file transfer procedure. After validation by Gireve, the Root CA will be integrated in Gireve's RCP by the authorized administrator.

---

### 3.4.2 Retrieving Root Certificates from the Gireve RCP

The PNCP service to retrieve root certificates from Gireve RCP is [Get RootCA certificates](https://gireve-apis.stoplight.io/docs/pncp/branches/main/p9ym7t2gvcd8e-get-root-ca-certificates). This service is paginated and reuses the OCPI pagination concepts (see [Section 2.4.4 — Pagination](./02-technical-guidelines.md#244-pagination)).

The following actors must retrieve Root Certificates from the Gireve RCP for the reasons described below:

- **The OEM** must retrieve and install in their EV's trust store the **V2G Root CAs**. This is necessary so the vehicle can:
  - Validate the SECC certificate chain when the EV is connected to the charging point.
  - Validate the CPS certificate chain in the SCCB before installing a contract certificate.

- **The eMSP** (or delegated system) must retrieve and install in their system the **OEM Root CAs** (V2G or OEM Root) used to generate the vehicle provisioning certificate. This is necessary for the eMSP to ensure the validation of the provisioning certificate chain before generating the CCB.

- **The CCP and PCP** must retrieve and install in their systems the **OEM Root CAs** (V2G or OEM Root) used to generate the vehicle provisioning certificate. This is necessary:
  - For the CCP to ensure the provisioning certificate chain validity when receiving a `certificationInstallationRequest` before sending back the appropriate SCCB.
  - For the PCP to ensure the provisioning certificate chain validity before adding the PC to the pool.

- **The CPO** must retrieve and install in their SECC or CSMS the **eMSP Root CAs** (V2G or MO Root) used to generate the contract certificates. This is necessary for the CPO to ensure the validity of the contract certificate chain passed by the EV to the charging point before allowing the charge.

- **The CPS** must retrieve and install in their system the **eMSP Root CAs** (V2G or MO Root) used to generate the contract certificates. This is necessary for the CPS to ensure the validity of the contract certificate chain contained in the bundle before signing it and turning it into a SCCB.

---

### 3.4.3 Being Notified that a Root Certificate has been Added or Removed from Gireve RCP

As detailed in [Section 3.5.2 — Webhook management](#352-webhook-management), there are three events associated with the addition or deletion of a Root Certificate in the Gireve RCP that can trigger a notification:

| Event | Description | Notified roles |
|---|---|---|
| `root.certificate.available` | New Root Certificate available in Gireve RCP | ALL |
| `root.certificate.expired` | Root Certificate removed from Gireve RCP because it has expired | ALL |
| `root.certificate.revoked` | Root Certificate removed from Gireve RCP because it is revoked | ALL |

Subscribers will only receive notifications for the root CAs relevant to them:

- **CPOs** will be notified for eMSP and V2G Root CAs.
- **eMSPs** will be notified for OEM Root CAs.
- **OEMs** will be notified for V2G Root CAs.

## 3.5 Notification Services Guidelines

Events occurring in the RCP, PCP, and CCP pools can impact operators connected to the platform. These events fall into the following categories:

- Data deposit in a pool
- Data withdrawal from a pool
- Data update in a pool
- Data deletion or certificate revocation in a pool
- Data / certificates nearing expiration date

A webhook notification system is available, allowing operators to subscribe to one or more event types. When a relevant event occurs that directly concerns them, a notification is sent.

---

### 3.5.1 How Does it Work?

A webhook is a real-time notification sent from one system to another via an HTTP POST request when specific events occur. The process is the following:

1. **Register** — The receiving system (the communication partner of an operator) provides a webhook endpoint URL to the sending system (the pool operator).
2. **Trigger** — An event happens in the sending system (e.g., new V2G Root CA available in RCP).
3. **Send** — The sending system pushes notification data (JSON payload) to the webhook URL.
4. **Respond** — The receiving system processes the data and performs actions (e.g., retrieval of the new V2G Root from the RCP).

---

### 3.5.2 Webhook Management

The following services are available for managing webhook configurations:

- [Create a webhook](https://gireve-apis.stoplight.io/docs/pncp/691fd2732a1d7-create-a-webhook) — Create a new webhook to receive notifications from the platform
- [Get all webhooks](https://gireve-apis.stoplight.io/docs/pncp/b6df95a558c0a-get-all-webhooks) — Fetch all webhooks configured in the platform by the requester
- [Get a webhook](https://gireve-apis.stoplight.io/docs/pncp/b47915205e4fa-get-a-webhook) — Fetch a specific webhook configured in the platform by the requester
- [Update a webhook](https://gireve-apis.stoplight.io/docs/pncp/9bd514facbdc5-update-a-webhook) — Update a webhook set by the requester to receive notifications from the platform
- [Delete a webhook](https://gireve-apis.stoplight.io/docs/pncp/branches/main/f9140af5727e0-delete-a-webhook) — Delete a webhook set by the requester to receive notifications from the platform

Each webhook is linked to one or more events (list of events in the section below). Operators can subscribe to as many events as they want. They can group these events together in one or more webhooks.

---

### 3.5.3 List of Events Triggering Webhooks

| Relevant for Role | Event Name | Description |
|---|---|---|
| ALL | `root.certificate.available` | A new Root Certificate is available in RCP |
| ALL | `root.certificate.expired` | Root Certificate removed from Gireve RCP because it has expired |
| ALL | `root.certificate.revoked` | Root Certificate removed from Gireve RCP because it has been revoked |
| eMSP | `emsp.provisionning.certificate.removed` | A SCCB linked to the eMSP has been removed from the CCP because the associated PC has been removed by the OEM |
| eMSP | `emsp.provisionning.certificate.revoked` | A SCCB linked to the eMSP has been removed from the CCP because the associated PC has been revoked by the OEM |
| eMSP | `emsp.provisionning.certificate.updated` | A SCCB linked to the eMSP has been removed from the CCP because the associated PC has been updated by the OEM |
| eMSP | `emsp.contract.certficate.delivered` | An OEM or a CPO have successfully retrieved from the CCP a SCCB linked to the eMSP for certificate installation |
| eMSP | `emsp.contract.certificate.expiring.soon` | A Contract Certificate (CC) is nearing expiration (within 50 days) and must be renewed to avoid service disruption |
| OEM | `oem.contract.certificate.available` | A new SCCB linked to one of the OEM PCs is available in the CCP |
| OEM | `oem.contract.certificate.revoked` | A CC linked to one of the OEM PCs has been revoked. It needs to be uninstalled by the OEM in the EV |
| OEM | `oem.provisionning.certificate.expiring.soon` | An OEM Provisioning Certificate (PC) is nearing expiration (within 180 days) and must be renewed |
| CPO | `cpo.secc.certificate.expiring.soon` | A SECC certificate is nearing expiration (within 15 days) and must be renewed |

---

### 3.5.4 How to Secure Your Webhook Endpoints?

To ensure that incoming messages to your webhook endpoint URLs are sent by Gireve and addressed to you, you can:

1. Check that the IP of the requester matches Gireve's output IP.
2. Check the `payload-signature` header of the webhook calculated from the secret.

**IP filtering:**

For all outgoing calls, Gireve presents a single fixed outgoing IP. This IP will be communicated to you during your PPROD and PROD onboarding. You should then whitelist it and only accept connections on your endpoint from this IP.

**Payload signature:**

Each webhook configuration is associated with a secret. This string is either defined by the operator during the creation or modification of their webhook or, by default, assigned by the Gireve Trust Platform. As a result, the secret is known only to the operator who set up the webhook and the Gireve Trust Platform.

When a notification is sent by Gireve, a `payload-signature` header is included. This header contains an HMAC-SHA256 signature (base64 format) of the notification payload, computed using the secret.

Upon receiving a notification, you should verify the signature by recalculating it using the secret. This ensures that the sender of the notification has knowledge of the secret, confirming that the message originates from the Gireve Trust Platform and is intended for you.

**Example — RCP notification payload** *(format described in Stoplight [here](https://gireve-apis.stoplight.io/docs/pncp))*:

```json
{
  "timestamp": "2024-10-17T19:48:14.369Z",
  "data": {
    "type": "V2G",
    "serialid": "279542570363453924971",
    "subjectdn": "DC=V2G,CN=Gireve-PKI 15118-2 V2G RootCA 1,O=Gireve,C=FR"
  },
  "eventtype": "root.certificate.available",
  "iso15118version": "urn:iso:15118:2:2013:MsgDef"
}
```

The HMAC-SHA256 `payload-signature` value calculated for this notification with the secret `!1234!` is:

`mOW99fmtFHRdU7V2TuE6EHXgZVqE037NGfk9JJMiOw=`

## 3.6 CPO Services Guidelines

### 3.6.1 Installation of All Necessary Certificates in the Charging Point

#### 3.6.1.1 Create a SECC Certificate and Retrieve Associated SubCAs

The services for creating a SECC certificate and retrieving its associated SubCAs are described in [Section 3.3.2 — Leaf certificate enrolment and obtention of linked SubCAs](#332-leaf-certificate-enrolment-and-obtention-of-linked-subcas) of this document.

As stated there, it is important to note that the CPO must install the SubCAs associated with the SECC Leaf Certificate in the SECC in order to pass the SECC certificate chain to the EV, to allow the EV to authenticate the SECC.

#### 3.6.1.2 Retrieve eMSP Root Certificates from Gireve RCP

CPOs must retrieve and install **eMSP Root CAs** (V2G or MO Root) in the trust store of their **SECC or CSMS**. These Root CAs are the basis for contract certificate validation. Ensuring their presence allows the CPO to verify the contract certificate chain presented by the EV before authorizing a charging session.

The service to retrieve eMSP Root certificates is [Get RootCA certificates](https://gireve-apis.stoplight.io/docs/pncp/branches/main/p9ym7t2gvcd8e-get-root-ca-certificates) as described in [Section 3.4.2 — Retrieving root certificates from the Gireve RCP](#342-retrieving-root-certificates-from-the-gireve-rcp).

***

### 3.6.2 SECC Certificate Lifecycle Management

#### 3.6.2.1 SECC Certificate Renewal

Please refer to [Section 3.3.3 — Renewal of a leaf certificate](#333-renewal-of-a-leaf-certificate) of this document.

#### 3.6.2.2 SECC Certificate Expiration Notification

To prevent service disruptions, Gireve proactively monitors SECC certificates (standard lifetime: 90 days). A webhook notification (`cpo.secc.certificate.expiring.soon`) is automatically triggered **15 days before the certificate expires**. CPOs are strongly advised to subscribe to this event to automate their certificate renewal process.

To prevent spam and ensure operators receive one alert per expiring certificate, Gireve flags the certificate internally as soon as the event is successfully emitted to the notification queue. For more information about notifications, please refer to [Section 3.5](#35-notification-services-guidelines) of this document.

#### 3.6.2.3 SECC Certificate Revocation

Please refer to [Section 3.3.4 — Revocation of a leaf certificate](#334-revocation-of-a-leaf-certificate) of this document.

***

### 3.6.3 Install the Contract Certificate in the Vehicle

There are two workflows for installing a contract certificate in a vehicle: installation via the CPO and installation via the OEM. Here, we focus on the installation of contract certificates by the CPO.

This is the workflow of installing a contract certificate on a vehicle via the EVCC/SECC link as described in [Section 5.3.5 of APPENDIX 2](#535-the-ev-user-plugs-its-iso-15118-ready-vehicle-on-an-iso-15118-ready-charging-point).

In the context of this use-case, the CPO system has to get the valid contract certificate from the CCP before sending it to the charging station. The PNCP service for this feature is [Get SCCBs by Certificate-Installation-Request](https://gireve-apis.stoplight.io/docs/pncp/m3c3u5t2098hj-get-scc-bs-by-certificate-installation-request).

> **📝 Note:** This service can be used as well by OEM when they handle the contract certificate in their vehicles themselves.

> **📝 Note 2:** A summary at the end of this section recapitulates the most important information about this service.

#### Input and Output Parameters Description

This service takes in parameters `iso_15118_version` and the `certificate_installation_req`.

The `iso_15118_version` parameter determines which version of the ISO-15118 protocol is installed on the vehicle. Gireve currently supports ISO 15118-2 and is working on integrating ISO 15118-20. The only accepted input for this parameter at the moment is the value corresponding to the ISO-15118-2 version which is: `"urn:iso:15118:2:2013:MsgDef"`.

The `certificate_installation_req` is the contract certificate installation request issued by the EV in the ISO-15118-2 format and is EXI and Base64 encoded. It consists of:

- A body containing the vehicle's provisioning certificate and the list of the V2G Root CAs that the vehicle trusts.
- A header containing a session ID and the body signature generated by the private key of the provisioning certificate.

The response to a certificate installation request is, as the name implies, a certificate installation response (`certificate_installation_res` parameter in the service response). It is the official format described by the ISO-15118-2 standard for carrying the SCCB in the vehicle (it can be interpreted by the EV). It is also EXI and Base64 encoded.

#### Filtering and Selection of SCCBs

When receiving a certificate installation request, Gireve performs a series of controls (detailed below) and then retrieves the valid SCCBs in its CCP which are:

1. Associated with the vehicle PCID extracted from the PC provided in the certificate installation request.
2. Signed by a non-expired CPS leaf certificate issued from a V2G Root CA trusted by the EV (meaning that V2G Root was part of the list of Root Certificates passed in the certificate installation request).
3. Containing a Contract Certificate with a validity period not expired.

If one or more matching SCCBs are found, they are then encoded in a certificate installation response format, and the same session ID provided in the request is set in the response.

All returned certificate installation responses are paired with an `owner_operator` that identifies the eMSP managing the contract certificate using country code and party id (eMI3 operator ID). This information is important because in ISO-15118-2, **only one contract certificate can be installed in the EV at a time**. If multiple SCCBs are compatible with the request and returned, the CPO selects one to send to the vehicle. The CPO may then prefer to install one of those generated by an eMSP with whom they have a valid roaming contract.

#### Description of the Controls in Place

As mentioned, when receiving a certificate installation request, Gireve performs all the necessary checks as described in the ISO-15118 standard:

- Conformity of the certificate installation request format.
- Validity of the provisioning certificate provided in the request (certificate chain validation, expiration check, revocation check).
- Validity of the signature in the request.

It is also possible that Gireve does not have any valid SCCBs associated with this PCID and the targeted V2G Root certificates. If no data is available or if a check fails, the service response will contain the attributes of a standard error response format with a specific `status_code` and a `status_message` describing the nature of the problem (see [Section 2.4.2 — Response format](#242-response-format)). In addition, the response will also include a certificate installation response (that can be forwarded to the EVCC) of minimum size made of arbitrary XSD-compliant values and a `ResponseCode` set to one of the values defined in the ISO-15118 standard according to the problem:

- `FAILED_SignatureError` — signature of the request invalid
- `FAILED_SequenceError` — impossible to decode the EXI request
- `FAILED_CertChainError` — no certificate chain linked to the PC
- `FAILED_CertificateExpired` — PC expired
- `FAILED_CertificateRevoked` — PC revoked
- `FAILED_NoCertificate_Available` — no SCCBs linked to the PCID and the targeted V2G Root CAs

Here is an example of a response when the provisioning certificate in the request has been revoked:

```json
{
    "data": [
        {
            "certificate_installation_res": "gJgCDEyMzQ1Njc4BFDGRAJgSATAgK0shiATAArSyGYBMACtLIZBAxMjM0NTY3ODlBQkNERQA"
        }
    ],
    "status_code": 2018,
    "status_message": "An element in the Provisioning Certificate chain is revoked",
    "timestamp": "2023-10-24T13:31:14Z"
}
```

This is the decoded certificate installation response (`certificate_installation_res`) from the previous error:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<ns6:V2G_Message xmlns:ns6="urn:iso:15118:2:2013:MsgDef"
  xmlns:ns5="http://www.w3.org/2000/09/xmldsig#"
  xmlns:ns7="urn:iso:15118:2:2013:MsgBody"
  xmlns:ns2="urn:iso:15118:2:2010:AppProtocol"
  xmlns:ns4="urn:iso:15118:2:2013:MsgDataTypes"
  xmlns:ns3="urn:iso:15118:2:2013:MsgHeader">
  <ns6:Header>
    <ns3:SessionID>3132333435363738</ns3:SessionID>
    <ns3:Notification>
      <ns4:FaultCode>UnknownError</ns4:FaultCode>
    </ns3:Notification>
  </ns6:Header>
  <ns6:Body>
    <ns7:CertificateInstallationRes>
      <ns7:ResponseCode>FAILED_CertificateRevoked</ns7:ResponseCode>
      <ns7:SAProvisioningCertificateChain>
        <ns4:Certificate>MA==</ns4:Certificate>
      </ns7:SAProvisioningCertificateChain>
      <ns7:ContractSignatureCertChain>
        <ns4:Certificate>MA==</ns4:Certificate>
      </ns7:ContractSignatureCertChain>
      <ns7:ContractSignatureEncryptedPrivateKey ns4:Id="id1">MA==</ns7:ContractSignatureEncryptedPrivateKey>
      <ns7:DHpublickey ns4:Id="id3">MA==</ns7:DHpublickey>
      <ns7:eMAID ns4:Id="id2">123456789ABCDE</ns7:eMAID>
    </ns7:CertificateInstallationRes>
  </ns6:Body>
</ns6:V2G_Message>
```

#### To Summarize

- Gireve currently supports ISO-15118-2 certificate installation requests. The value to set in the `iso_15118_version` parameter is `"urn:iso:15118:2:2013:MsgDef"`.
- In ISO-15118-2, only one contract certificate can be installed in the EV at a time. If multiple `certificate_installation_res` are returned, the CPO selects one to send to the vehicle. The `owner_operator` parameter can help in this selection by identifying the eMSP that manages the contract certificate contained in the `certificate_installation_res`. The CPO can then prioritize the eMSPs with which they have a valid roaming contract.
- Gireve performs all the necessary checks as described in the ISO-15118 standard. If no data is available or if a check fails, the service response will be a standard error response with a specific `status_code` and a `status_message` describing the nature of the problem. In addition, as required by the ISO-15118 standard, the response will also include a `certificate_installation_res` of minimum size consisting of arbitrary XSD-compliant values and a `ResponseCode` with the correct value according to the problem.

***

### 3.6.4 Consultation of Enrolled SECC Certificate

CPOs may use the Certificate Consultation Services to review or export the SECC certificates they have enrolled through the platform. This can help technical teams reconcile issued certificates with their internal inventories and support audit activities.

Please refer to [Section 3.3.5 — Certificate consultation services](#335-certificate-consultation-services) of this document.

***

### 3.6.5 Metadata Usage for CPOs

CPOs can associate metadata with certificates at the time of creation or renewal, as explained in [Section 3.3.2](#332-leaf-certificate-enrolment-and-obtention-of-linked-subcas) and [Section 3.3.3](#333-renewal-of-a-leaf-certificate). This metadata can then be used to filter, search, and export targeted subsets of SECC certificates more efficiently in the Certificate Consultation Services.

***

### 3.6.6 Manage Roaming Authentication Workflow with eMAIds

Gireve supports eMIP and OCPI roaming protocols. Roaming authentication with eMAId uses the same workflows and web services as with a classic RFID token authentication.

End users, the eMSP customers, are identified in the eMSP's information system, which assigns them an authentication media that can be:

- An **RFID badge** carrying a number (called RFID-UID) for "classic" recharging.
- A **digital certificate** (the contract certificate) to be installed in the vehicle for Plug&Charge. This certificate carries an identifier called **eMAID** (e-Mobility Account Identifier).

In terms of systems, these identifiers are contained in objects called **authentication data** in eMIP and **tokens** in OCPI.

#### 3.6.6.1 Manage Roaming Authentication Workflow with eMAIds in OCPI

In OCPI, real-time authorization requests are processed via the [**POST Token « authorize »**](https://github.com/ocpi/ocpi/blob/release-2.1.1-bugfixes/mod_tokens.md#222-post-method) flow between the CPO and the Gireve Roaming platform.

In all OCPI token services, tokens for Plug&Charge contain the **eMAID** value in the `auth_id` field and must be associated with the **`OTHER`** type.

The OCPI protocol description for version 2.1.1 is available [here](https://github.com/ocpi/ocpi/tree/release-2.1.1-bugfixes) and the Gireve OCPI implementation guide is available [here](https://github.com/CNX-GIREVE/GIREVE_Tech_OCPI_V2.1.1).

#### 3.6.6.2 Manage Roaming Authentication Workflow with eMAIds in eMIP

In eMIP, real-time authorization requests are processed via the **eMIP_ToIOP_GetServiceAuthorisation** flow between the CPO and the Gireve Roaming platform.

In all eMIP authentication data services, authentication data for Plug&Charge contains the **eMAID** value in the `userId` field and must be associated with the **`EMP-SPEC`** `userIdType`.

The Gireve eMIP complete implementation guide is available [here](https://www.gireve.com/wp-content/uploads/2022/09/Gireve_Tech_eMIP-V0.7.4_ProtocolDescription_1.0.14-en.pdf).

## 3.7 eMSP Services Guidelines

### 3.7.1 Creation of a contract certificate and making it available in CCP for installation in EV

![Diagram of the contract certificate creation and CCP storage workflow](images/media/image9.png)

To create a Contract Certificate and make it available for installation in the target EV, the eMSP must complete or delegate the following actions:

1. Generate a new key pair for the future Contract Certificate in a secure environment.
2. Obtain a Contract Certificate associated to that key pair issued from an eMSP SubCA2.
3. Create a Contract Certificate bundle (CCB). Creating the bundle requires the following additional tasks to be performed beforehand:
   1. Retrieve the OEM's Root certificates from an RCP to add them in the system trust store to validate the OEM Provisioning Certificate chains. This needs to be done regularly as new OEMs Root Certificates might be available over the time.
   2. Fetching the PC linked to the eMSP's Customer's PCID from a PCP.
   3. Verifying the validity of the retrieved PC before generating the bundle (certificate chain valid and linked to a trusted OEM Root. Certificate not expired and not revoked).
4. Sign the Contract Certificate Bundle with the private key of a CPS leaf certificate (it becomes then a SCCB). This action requires to perform the following additional tasks beforehand:
   1. Retrieve the eMSP's Root certificates from an RCP and add them in the system trust store to validate the eMSP Contract Certificate chains from the CCBs. This needs to be done regularly as new eMSP Root Certificates might be available over the time. If the entity issuing the Contract Certificate (step 2) and signing the bundle (step 4) is the same actor, this step might be optional.
   2. Generate a new key pair for the future CPS Leaf certificate in a secure environment.
   3. Obtain a Leaf CPS Certificate associated with that key pair issued from a CPS SubCA2.
   4. Verifying the validity of the CC from the CCB before signing it (certificate chain valid and linked to a trusted eMSP Root. Certificate not expired and not revoked).
   5. Make the SCCB available in a CCP so it can be fetched and installed by OEMs or CPOs.

Gireve strongly recommends to the EMSP the use of the [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp), also known as **Build Sign Store**, which covers all the required actions for the EMSP. This service minimizes the EMSP's implementation requirements and eliminates the need to invest in an expensive PKI infrastructure with a Certificate Manager, HSMs, CRL distribution points or OCSP responders.

Gireve provides several services from the PNCP protocol that perform one, multiples or all of these actions.

#### 3.7.1.1 Creation of a contract certificate and a contract certificate bundle

> **📝 Note:** The actions "Generate a new Key Pair" (step 1) and "Create Contract Certificate Bundle" (step 3) must be performed by the same actor. The creation of the CCB requires the possession of the Contract Certificate private key and this private key must never circulate unencrypted and outside the specific installation workflows defined in the ISO-15118 standard.

##### 3.7.1.1.1 Option 1 – Gireve generates the CC key pair and the CCB

Gireve exposes the PNCP services [Create a CC and build CCB](https://gireve-apis.stoplight.io/docs/pncp/wstbyvxtxtnuy-create-a-cc-and-build-ccb) and [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp). Both services take care of the creation of a new certificate key pair, the issuance of a contract certificate and the creation of the Contract Certificate Bundle associated to the CC and targeted PC. The second service will also sign the bundle and store it in Gireve PCP.

The eMSP must simply here specify the eMAId (E-Mobility Account Identifier) that will be applied to the Common Name of the issued Contract Certificate and the PCID corresponding to the vehicle (the PCID of an EV is normally communicated by the OEM to the buyer. It must be specified to the eMSP when activating the plug and charge on an EV).

The eMSP may also choose to specify a specific `certificateprofile` that determines exactly which SuBCA2 will issue the Contract Certificate, the validity period of that certificate and how the Subject DN of that certificate will be set (see [Section 3.3.1](#certificates-profiles-definition-and-usage) on certificate profiles). This parameter is optional. If not set, the default `certificateprofile` configured for this operator during the technical onboarding will be used.

> **📝 Note:** These two services also allow eMSP to add optional `metadata` object in the request body when creating or renewing the CC and associated CCB. This metadata enables the association of simple business classification data with the created certificate record in Gireve systems. It is not embedded in the certificate itself. This metadata can then be used to simplify reporting, filtering, and internal monitoring. For more information, see [Section 3.7.12](#metadata-usage-for-emsps).

##### 3.7.1.1.2 Option 2 – The eMSP generates the CC key pair and the CCB

After the creation of a key pair on its own, the eMSP can choose to obtain its Contract Certificate by using the PNCP enrolment services exposed by Gireve as detailed in [Section 3.3.2](#leaf-certificate-enrolment-and-obtention-of-linked-subcas).

As a reminder, when creating a new key pair for a contract certificate, the eMSP must ensure the key pair is based on `secp256r1` elliptic curve as required by the ISO-15118-2 standard. Gireve will verify this and reject the enrolment request if it is not applied.

Remember that private keys should be protected. eMSP is bound to its MO-Root by an agreement regarding security and its MO-Root's certificate policy.

To be able to generate the bundle the eMSP must:
- Be able to fetch the PC linked to the eMSP's Customer's PCID from a PCP.
- Retrieve the OEM's Root certificates from an RCP to be able to validate the PC.

As a PCP, Gireve offers a PNCP service to retrieve a PC from a PCID. It is described in [Section 3.7.6](#retrieve-an-oem-pc-from-a-pcid).

As an RCP, Gireve offers a PNCP service to retrieve the OEM Root Certificates. It is described in [Section 3.7.8](#retrieve-oem-root-certificate-in-gireve-rcp).

The eMSP can then create the Contract Certificate Bundle using the public key of the Provisioning Certificate.

> **📝 Note:** Once the bundle has been generated, the eMSP must remove the unencrypted private key from its system for security reasons and delete it from any storage.

> **📝 Note 2:** The unencrypted private key must never be logged or traced.

#### <span id="Signingthecontract" class="anchor"></span>3.7.1.2 Signing the contract certificate bundle

Gireve also takes on the role of Certificate Provisioning Service and presents PNCP services that sign the contract certificate bundle in their process.

The signing of the Contract Certificate Bundle by Gireve is included in the [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp) service process. It also makes available the SCCB into Gireve's CCP.

Gireve also offers two services that take a Contract Certificate Bundle as an input and return a Signed Contract Certificate Bundle in the response: [Sign a CCB and return SCCB](https://gireve-apis.stoplight.io/docs/pncp/b9rrqqo23czpv-sign-a-ccb-and-return-sccb) and [Sign a CCB and store SCCB in CCP](https://gireve-apis.stoplight.io/docs/pncp/6ed4bedec7e9d-sign-a-ccb-and-store-sccb-in-ccp). The latter one also makes available the SCCB into Gireve's CCP.

These two services require to specify the whole provisioning certificate used to generate the CCB.

If a EMSP doesn't use Gireve Certificate enrollment services to generate its Contract Certificates, it must make its EMSP Root CA available in Gireve's RCP as described in [Section 3.7.5](#making-the-emsp-root-certificate-available-in-an-rcp).

Gireve recommends using the [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp). This is much more efficient and simpler than combining the [Create a CC and build CCB](https://gireve-apis.stoplight.io/docs/pncp/wstbyvxtxtnuy-create-a-cc-and-build-ccb) service with the two possible signing services.

#### 3.7.1.3 Storing the SCCB in Gireve CCP

Adding an SCCB to the Gireve CCP is handled by two services:

- [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp), which also generates the key pair, the Contract Certificate, the CCB and the SCCB beforehand.
- [Sign a CCB and store SCCB in CCP](https://gireve-apis.stoplight.io/docs/pncp/6ed4bedec7e9d-sign-a-ccb-and-store-sccb-in-ccp), which takes as an input a CCB and the linked OEM provisioning certificate and returns an SCCB after storing it in Gireve CCP.

### 3.7.2 Contract certificate renewal

If the eMSP delegates to Gireve the creation of the contract certificate key pairs, the issuance of the contract certificates and the creation of the Contract Certificate Bundles, it uses then either the [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp) or the [Create a CC and build CCB](https://gireve-apis.stoplight.io/docs/pncp/wstbyvxtxtnuy-create-a-cc-and-build-ccb) service. To renew a contract certificate, the eMSP simply calls again one of these services and specifies the eMAId of the contract certificate it wants to renew.

If the eMSP used Gireve certificates enrolment services, [Enroll a certificate PNCP](https://gireve-apis.stoplight.io/docs/pncp/afc0baf98a6e1-enroll-a-certificate-pncp) or [Enroll a certificate EST protocol](https://gireve-apis.stoplight.io/docs/pncp/7dd53d389f659-enroll-a-certificate-est-protocol), to obtain its Contract Certificates (meaning it generates itself the key pair and creates itself the bundle), it can use the same enrolment services for its certificate renewal as described in [Section 3.3.3](#renewal-of-a-leaf-certificate).

> **📝 Note:** As explained in [Section 3.7.1.1.1](#option-1-gireve-generates-the-cc-key-pair-and-the-ccb) and [Section 3.3.3](#renewal-of-a-leaf-certificate), these PNCP services allow the eMSP to include an optional `metadata` object in the request body when renewing a leaf certificate. This metadata enables the association of simple business classification data with the created certificate record in Gireve systems. It is not embedded in the certificate itself. This metadata can then be used to simplify reporting, filtering, and internal monitoring. For more information, see [Section 3.7.3](#consultation-of-enrolled-Contract-Certificate).

### 3.7.3 Consultation of enrolled Contract Certificate

eMSPs may use the Certificate Consultation Services to review or export the Contract Certificates they have enrolled through the platform. These services are intended for consultation and reporting use cases and do not replace the dedicated lifecycle operations used to create, renew or revoke certificate.

In addition, eMSP can associate metadata with certificates, as explained in the certificate creation [Section 3.7.1.1.1](#option-1-gireve-generates-the-cc-key-pair-and-the-ccb) and renewal [Section 3.7.2](#contract-certificate-renewal), this metadata can be provided when enrolling or renewing a certificate. It can then be used to filter, search, and export targeted subsets of certificates more efficiently.

### 3.7.4 Contract certificate expiration notification

Contract Certificates have a standard lifetime of 2 years. To assist eMSPs in anticipating renewals, Gireve triggers a webhook notification `emsp.contract.certificate.expiring.soon` 50 days prior to the certificate's expiration date. Operators must subscribe to this event to ensure continuous roaming authentication for their customers.

To prevent spam and ensure operators receive one alert per expiring certificate, Gireve flags the certificate internally as soon as the event is successfully emitted to the notification queue. For more information about notifications, please refer to [Section 3.5](#notification-services-guidelines).

### 3.7.5 Contract certificate revocation and removal from CCP

<span id="Contractcertificaterevocation" class="anchor"></span>

It is important to distinguish the contract certificate revocation and the contract certificate deactivation in the Gireve CCP (acting the revocation of a CC in Gireve CCP). Depending on the case, some of the PNCP services exposed by Gireve will take only one action or both actions.

![Diagram of contract certificate revocation and CCP removal workflow](images/media/image10.png)

#### 3.7.5.1 Option 1 – Gireve created the contract certificate and the bundle

![Diagram of revocation workflow – Option 1](images/media/image11.png)

In this case, the eMSP used either the [Create CC, build sign the CCB and store in CCP](https://gireve-apis.stoplight.io/docs/pncp/yuu57zepf5oaf-create-cc-build-and-sign-the-ccb-and-store-in-ccp) or the [Create a CC and build CCB](https://gireve-apis.stoplight.io/docs/pncp/wstbyvxtxtnuy-create-a-cc-and-build-ccb) service to generate the contract certificate and the bundle.

To revoke a contract certificate issued this way the eMSP must call either [Revoke a CC](https://gireve-apis.stoplight.io/docs/pncp/fxqkvxwoijkv8-revoke-a-cc) or [Deactivate an eMAId](https://gireve-apis.stoplight.io/docs/pncp/ea6ea0c1989ac-deactivate-an-e-ma-id). The first service will target a specific Contract Certificate identified by the pair serial number (in decimal value) and Issuer DN, the second service will target all Contract Certificates linked to the requesting eMSP and associated to the eMAId set in the request parameter. All targeted contract certificates are first revoked by the Certificate Authority that issued them. Then the eventual SCCBs containing these now revoked contract certificates are deleted from Gireve's CCP.

> **📝 Note:** After a revocation the CA needs to update its CRL and add the new revoked certificate. This update is done periodically (every few hours up to a day). Therefore, if you revoke a certificate and immediately check the revocation status with an OCSP call or by downloading the CRL, the certificate will still be displayed as valid until the CRL is updated.

> **📝 Note 2:** Services [Revoke a certificate](https://gireve-apis.stoplight.io/docs/pncp/k1cwhljgcbw3z-revoke-a-certificate) and [Act revocation of a CC in CCP](https://gireve-apis.stoplight.io/docs/pncp/rkknin0bix89l-act-revocation-of-a-cc-in-ccp) are to be specifically applied on Contract Certificates issued by the generic enrolment services (see section below). If called in this case the request will be rejected.

#### 3.7.5.2 Option 2 – The eMSP generated the CC key pair and the certificate bundle

![Diagram of revocation workflow – Option 2](images/media/image12.png)

We assume here that the eMSP used Gireve certificates enrolment services, [Enroll a certificate PNCP](https://gireve-apis.stoplight.io/docs/pncp/afc0baf98a6e1-enroll-a-certificate-pncp) or [Enroll a certificate EST protocol](https://gireve-apis.stoplight.io/docs/pncp/7dd53d389f659-enroll-a-certificate-est-protocol), to obtain its Contract Certificates. If the eMSP didn't enrolled its CC with Gireve's PKI services, Gireve then cannot revoke the CC as the revocation can only be performed by the Certificate Authority that issued the certificate.

In this case, the service to use to revoke a Contract Certificate is [Revoke a certificate](https://gireve-apis.stoplight.io/docs/pncp/k1cwhljgcbw3z-revoke-a-certificate) as described in [Section 3.3.4](#revocation-of-a-leaf-certificate). **This service will only perform the revocation action.** The eventual SCCB in Gireve's CCP containing the revoked certificate will remain in the CCP.

After revoking a contract certificate, to act this revocation, meaning to remove all the SCCB from Gireve CCP that are linked to a revoked certificate, the eMSP must call either [Act revocation of a CC in CCP](https://gireve-apis.stoplight.io/docs/pncp/rkknin0bix89l-act-revocation-of-a-cc-in-ccp) or [Deactivate an eMAId](https://gireve-apis.stoplight.io/docs/pncp/ea6ea0c1989ac-deactivate-an-e-ma-id). The first service will target a specific Contract Certificate identified by the pair serial number (in decimal value) and Issuer DN, the second service will target all Contract Certificates linked to the requesting eMSP and associated to the eMAId set in the request parameter.

> **📝 Note:** The [Deactivate an eMAId](https://gireve-apis.stoplight.io/docs/pncp/ea6ea0c1989ac-deactivate-an-e-ma-id) service will not also handle the revocation of the targeted CCs as described in the previous workflow where Gireve generated the CC and the bundle itself. It is essential to call the [Revoke a certificate](https://gireve-apis.stoplight.io/docs/pncp/rs5onohqhyyi2-revoke-a-certificate) service on that certificate beforehand.

> **📝 Note 2:** Service [Revoke a CC](https://gireve-apis.stoplight.io/docs/pncp/fxqkvxwoijkv8-revoke-a-cc) is to be specifically applied in the previous workflow where Gireve generated the CC and the bundle itself. If called in this case the request will be rejecte

### 3.7.6 Making the eMSP Root certificate available in an RCP

<span id="section-4" class="anchor"></span>

If an eMSP manages its own Root Certificate, it must make it available in Gireve's RCP. This is necessary because the CPS and CPO need this root certificate to validate the eMSP's Contract Certificates. Without this information, these actors will reject the eMSP's Contract Certificates. This includes Gireve that acts as a CPS when the eMSP calls the PNCP signing services as described in [Section 3.7.1.2](#signing-the-contract-certificate-bundle).

For all these reasons, all eMSP connected to Gireve's Trust platform are required to make their root certificates available in Gireve's RCP.

The process for making available a Root Certificate in Gireve's RCP is described in [Section 3.4.1](#making-a-root-certificate-available-in-the-gireve-rcp).

### 3.7.7 Retrieve an OEM PC from a PCID

<span id="RetrieveanOEM" class="anchor"></span>

If the eMSP generates the contract certificate bundle itself and does not delegate the task to Gireve, then the eMSP will need to retrieve the PC that is associated to its customer PCID in a PCP and to verify the validity of that PC before using it to generate the bundle.

As a PCP, Gireve presents the PNCP service [Get PC by PCID](https://gireve-apis.stoplight.io/docs/pncp/yemwrjvxxsjoj-get-pc-by-pcid) that will take as param input a PCID and return the full provisioning certificate linked to that PCID including its certificate chain.

> **📝 Note:** Gireve's PCP only return active provisioning certificate. Meaning they are not expired or revoked.

> **📝 Note 2:** If multiple active PCs share the same PCID (in case of a PC renew during the transition period), Gireve PCP will send back the most recent version of the PC.

### 3.7.8 Check an OEM PC from a PCID

If an eMSP delegates the creation of the contract certificate and bundle to Gireve, they may first want to verify whether the PCID provided by their customer is valid and associated with a PC in Gireve's PCP.

To facilitate this, Gireve, as a PCP, offers the PNCP service [Check a PCID in PCP](https://gireve-apis.stoplight.io/docs/pncp/), a streamlined and faster alternative to [Get PC by PCID](https://gireve-apis.stoplight.io/docs/pncp/yemwrjvxxsjoj-get-pc-by-pcid). This service takes a PCID as input and returns confirmation if the PCID is valid and linked to a PC in Gireve's PCP; otherwise, it provides an error response.

### 3.7.9 Retrieve OEM Root Certificate in Gireve RCP

<span id="RetrieveOEMRoot" class="anchor"></span>

If the eMSP generates the contract certificate bundle itself and does not delegate the task to Gireve, then the eMSP will need to retrieve the PC that is associated to its customer PCID in a PCP and verify the validity of that PC before using it to generate the bundle.

To verify the validity of the PC, the eMSP must retrieve the OEM Root Certificate from Gireve RCP like Gireve's one.

The service to fetch Root Certificates from Gireve RCP is described in [Section 3.4.2](#retrieving-root-certificates-from-the-gireve-rcp).

### 3.7.10 Be notified if SCCB is removed or retrieved from Gireve CCP

As explained in [Section 3.5](#notification-services-guidelines), the eMSP can receive webhook notifications about the following events related to their SCCBs:

- `emsp.provisionning.certificate.removed`
- `emsp.provisionning.certificate.revoked`
- `emsp.provisionning.certificate.updated`
- `emsp.contract.certificate.delivered`

The first three events indicate that the SCCB has been removed from the Gireve CCP due to the OEM deleting, revoking, or updating the Provisioning Certificate associated with the Contract Certificate.

In such cases, if the Contract Certificate has not yet been installed in the vehicle, the eMSP must regenerate the data using the new Provisioning Certificate.

The last event notifies the eMSP that one of their SCCBs has been retrieved by an OEM or a CPO for Contract Certificate installation in the EV. However, retrieval does not guarantee that the installation was successfully completed afterward.

### 3.7.11 Manage roaming authentication workflow with eMAIds

Gireve supports eMIP and OCPI roaming protocols. Roaming authentication with eMAId uses the same workflows and web services as classic RFID token authentication.

End users — the eMSP customers — are identified in the eMSP's information system, which assigns them an authentication media that can be:

- An RFID badge carrying a number called RFID-UID for classic recharging.
- A digital certificate — the Contract Certificate — to be installed in the vehicle for Plug&Charge. This certificate carries an identifier called eMAID (E-Mobility Account Identifier). In terms of systems, these identifiers are contained in objects called "authentication data" in eMIP and "tokens" in OCPI.

#### 3.7.11.1 Manage roaming authentication workflow with eMAIds in OCPI

In OCPI, real-time authorization requests are processed via the [POST Token authorize](https://github.com/ocpi/ocpi/blob/release-2.1.1-bugfixes/mod_tokens.md#222-post-method) flow between Gireve Roaming platform and the eMSP.

In all OCPI token services, tokens for Plug&Charge contain the eMAID value in the `authid` field and must be associated with the **OTHER** type.

The OCPI protocol description for 2.1.1 version is available [here](https://github.com/ocpi/ocpi/tree/release-2.1.1-bugfixes) and Gireve OCPI implementation guide is available [here](https://github.com/CNX-GIREVE/GIREVE_Tech_OCPI_V2.1.1).

#### 3.7.11.2 Manage roaming authentication workflow with eMAIds in eMIP

In eMIP, real-time authorization requests are processed by `eMIPFromIOPGetServiceAuthorisation` between the Gireve Roaming platform and the eMSP.

In all eMIP authentication data services, authentication data for Plug&Charge contain the eMAID value in the `userId` field and must be associated with the **EMP-SPEC** `userIdType`.

Gireve eMIP complete implementation guide is available [here](https://www.gireve.com/wp-content/uploads/2022/09/Gireve_Tech_eMIP-V0.7.4_ProtocolDescription_1.0.14-en.pdf).

## 3.8 OEM services guidelines

### 3.8.1 Installation of all necessary certificates in the EV

#### 3.8.1.1 Obtain a PC certificate and its linked SubCAs

The services for creating a Provisioning Certificate and retrieving its associated SubCAs are described in [Section 3.3.2](#leaf-certificate-enrolment-and-obtention-of-linked-subcas) of this document.

As stated there, it is important to note that the OEM must include the associated SubCAs along with the Provisioning Certificate when making it available in a PCP to allow the PCP to validate the PC before integrating it, and later for the eMSP to validate the PC again before using it in a Contract Certificate Bundle.

#### 3.8.1.2 Retrieve the list of V2G Root certificates from Gireve RCP

The OEM needs to retrieve and install in their EV's trust store the V2G Root CAs. This is necessary so the vehicle can validate:

- The SECC certificate chain when the EV is connected to the charging point.
- The CPS certificate chain in the SCCB before installing a contract certificate.

As some V2G Root CAs might be added over time in the RCP, the OEM may call the RCP regularly to update the trust store of their EVs and ensure their full compatibility with the actors of the Plug&Charge ecosystem.

The service to retrieve V2G Root certificates is [Get RootCA certificates](https://gireve-apis.stoplight.io/docs/pncp/branches/main/p9ym7t2gvcd8e-get-root-ca-certificates) as described in [Section 3.4.2](#retrieving-root-certificates-from-the-gireve-rcp).

> To be notified if a V2G Root Certificate is made available or removed from Gireve RCP, the OEM should subscribe to the RCP notifications as described in Section 3.4.3.

#### 3.8.1.3 Install a contract certificate in the EV

There are two workflows for installing a contract certificate in a vehicle: installation via the CPO and installation via the OEM. Here, we focus on the installation of contract certificates by OEMs.

Gireve presents two PNCP services enabling OEMs to retrieve the Signed Contract Certificates Bundles (SCCB) which can be transmitted to the vehicle for Contract Certificate installation:

- [Get SCCBs by Certificate-Installation-Request](https://gireve-apis.stoplight.io/docs/pncp/m3c3u5t2098hj-get-scc-bs-by-certificate-installation-request)
- [Get SCCBs linked to PCID](https://gireve-apis.stoplight.io/docs/pncp/ke5ca08wey2mu-get-scc-bs-linked-to-pcid)

![table example](./images/media/image13.png)

##### Option 1: Using the Get SCCBs by Certificate-Installation-Request service

[Get SCCBs by Certificate-Installation-Request](https://gireve-apis.stoplight.io/docs/pncp/m3c3u5t2098hj-get-scc-bs-by-certificate-installation-request) is a service also used by CPOs to install a contract certificate in an EV. It retrieves the SCCBs associated to a `CertificateInstallationRequest` emitted by the EV. The SCCBs are returned in an ISO-15118 `CertificateInstallationResponse` format, in EXI and Base64 encoded. A detailed explanation of this service can be found in [Section 3.5.3](#install-the-contract-certificate-in-the-vehicle).

##### Option 2: Using the Get SCCBs linked to PCID service

An OEM can only pull the SCCBs linked to its own Provisioning Certificates from Gireve's CCP.

[Get SCCBs linked to PCID](https://gireve-apis.stoplight.io/docs/pncp/ke5ca08wey2mu-get-scc-bs-linked-to-pcid) is an OEM-only service. It retrieves the SCCBs associated with a PCID. The format of the SCCBs returned in the response can be selected and can be in JSON, in EXI or both.

**Selection of the Format:** The selection of the return format of the SCCB is done with the request query parameter `sccb_format` that can take the following values:

- `sccb_format=JSON` — This is the default value, meaning if the parameter is not present in the request, this format will be considered. With this configuration the SCCBs are returned in JSON in the response object `signed_contract_certificate_bundle`.
- `sccb_format=EXI` — With this configuration the SCCBs are returned in the response string `certificate_installation_res` in EXI format and Base64 encoded.
- `sccb_format=ALL` — With this configuration, each SCCB will be returned in EXI in the response string `certificate_installation_res` and in JSON in the response object `signed_contract_certificate_bundle`.

**Selection of a specific PC:** The PCID set as input parameter targets most of the time one PC. However, particularly in the case of a PC renewal, a PCID may target more than one active PC. For this reason, optional parameters `serial_number` (decimal format) and `issuer_dn` can also be set to ensure that only the SCCBs associated with a particular PC are pulled.

**Selection of a SCCB:** All returned SCCBs are paired with an `owner_operator` that identifies the eMSP managing the contract certificate using country code and party id (emi3 operator ID) pair.

### 3.8.2 PC certificate lifecycle management

#### 3.8.2.1 PC certificate renewal

The same enrolment services used to generate a PC can be used for a PC renewal (see [Section 3.3.3](#renewal-of-a-leaf-certificate)).

It is important to note that the renewal of a PC will not replace the old PC in Gireve's PCP. The update of a PC in Gireve PCP is performed by another service as described in [Section 3.8.2](#pc-certificate-lifecyle-management).

#### 3.8.2.2 PC certificate expiration notification

Provisioning Certificates have a long lifetime (typically 5 years). To help OEMs manage this lifecycle without service interruption, Gireve issues an early warning webhook notification `oem.provisionning.certificate.expiring.soon` 180 days before expiration. Upon receiving this event, OEMs should schedule the generation and provisioning of a new PC.

To prevent spam and ensure operators receive one alert per expiring certificate, Gireve flags the certificate internally as soon as the event is successfully emitted to the notification queue. For more information about notifications, please refer to [Section 3.5](#notification-services-guidelines) of this document.

#### 3.8.2.3 PC certificate revocation

Please refer to [Section 3.3.4](#revocation-of-a-leaf-certificate) of this document.

It is also important to note that a PC revocation will not automatically remove that PC from Gireve PCP nor deactivate the SCCBs linked to this PC's PCID. To do that, another action needs to be performed by the OEM: acting the revocation of the PC. This is detailed in [Section 3.7.3.3](#act-the-revocation-of-a-provisioning-certificate-in-gireve-pcp).

#### 3.8.2.4 Consultation of enrolled Provisioning Certificate

OEMs may use the Certificate Consultation Services to review or export the Provisioning Certificates they have enrolled through the platform. These services provide a self-service view of enrolled certificate data and can support operational follow-up or audit preparation.

#### 3.8.2.5 Classification of PC data with Metadata

OEMs may use metadata to classify the PC certificates they enroll through the platform according to their own operational criteria. This metadata can then be reused in [Certificate Consultation Services](#certificate-consultation-services) to find or export a targeted subset of certificates more easily.

### 3.8.3 Provision and administration of PCs in Gireve pools

#### 3.8.3.1 Making a provisioning certificate available in Gireve PCP

The service to make a provisioning certificate available in Gireve's PCP is [Add a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/wjijwh6psp6xl-add-a-pc-in-pcp).

The OEM must include in the request:

- The OEM leaf Provisioning Certificate
- The SubCAs (intermediate certificates)
- The ISO-15118 version
- The list of V2G Root certificates that are trusted by the EV

> **📝 Note:** The parameter `oldpcid` has to be specified only for updating a PC already included in Gireve PCP. The explanations on that process are detailed in the next section. We consider here that this parameter is not set by the requester.

The Provisioning Certificate and the SubCAs must be passed in Base64 PEM format without headers or line breaks. The SubCAs need to be ordered in the list from lowest certificate authority to highest certificate authority.

Gireve currently supports ISO 15118-2 certificates. The value to set in the `iso15118version` parameter is `urn:iso:15118:2:2013:MsgDef`.

The parameter `v2grootcakeyidentifierlist` contains the list of V2G Root Certificates trusted by the EV. The V2G Root CAs are identified by their `serialid` in decimal format and `issuerdn` equals to the subject since Root CAs are self-issued.

When Gireve receives an [Add a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/wjijwh6psp6xl-add-a-pc-in-pcp) request from the OEM, it will first validate the provisioning certificate — verifying that the certificate chain is associated to a trusted OEM Root CA and that the chain is valid (including expiration and revocation controls). Therefore, if the OEM has not communicated its OEM Root CA to Gireve in advance, all PCs from this OEM will be rejected. To do so, please see [Section 3.8.4](#make-an-oem-root-available-in-gireve-rcp).

> As specified in Section 10 of the VDE-AR-E 2802-100-1:2019-12 document, the PCID (common name of the PC) must begin with the OEM's World Manufacturer Identifier (WMI) code.
>
> When Gireve receives a PC from an OEM, it will verify that the WMI in the PCID exists and is correctly associated with the OEM submitting the PC. If this verification fails, the PC will be rejected.
>
> Additionally, if the request concerns the addition of a new PC (`old_pcid` is not provided) and the PCID of the new PC already matches an existing PC in Gireve's PCP, the request will also be rejected.

#### 3.8.3.2 Updating a provisioning certificate in Gireve PCP

Updating a PC in the Gireve PCP is also done with the [Add a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/wjijwh6psp6xl-add-a-pc-in-pcp) service. The difference with an insert is the use of the `oldpcid` property in the request body. In this case, all validity checks on the PC are also applied (verification of the chain, expiration and revocation).

Gireve supports the following cases:

- Updating a PC with a new version with the same PCID (`oldpcid` and new `pcid` are equal and correspond to a PC in Gireve's PCP).
- Update of a PC in the PCP by a new version with a different PCID (`oldpcid` corresponding to a PC in Gireve's PCP and new PCID not yet referenced in Gireve's PCP).

Gireve rejects the following case:

- Updating a PC in Gireve's PCP with a new version with a different PCID that also corresponds to an existing PC (different `oldpcid` and new PCID, new PCID already referenced in Gireve's PCP).

#### 3.8.3.3 Act the revocation of a provisioning certificate in Gireve PCP

<span id="Acttherevocation" class="anchor"></span>

After revoking an OEM Provisioning Certificate, the OEM must remove that PC from Gireve PCP and deactivate the SCCBs linked to this PC's PCID in Gireve CCP. This process is called "Acting the revocation of a Provisioning Certificate". To do so, the OEM must call the service [Act revocation of a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/9631ac36e5e6a-act-revocation-of-a-pc-in-pcp).

This service will target the Provisioning Certificate by identifying it with the serial number (in decimal value) and Issuer DN set in the request.

> **📝 Note:** Gireve does not check the revocation status of the Provisioning Certificate before performing the actions in the [Act revocation of a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/9631ac36e5e6a-act-revocation-of-a-pc-in-pcp) service. If the OEM has not revoked the Provisioning Certificate before and calls, the PC will be removed from Gireve PCP and the linked SCCBs will be deactivated from Gireve CCP anyway.

#### 3.8.3.4 Removing a provisioning certificate in Gireve PCP

The [Delete a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/2dze978qcx6ao-delete-a-pc-in-pcp) service enables an OEM to remove a Provisioning Certificate from Gireve's PCP. An OEM can only delete its own Provisioning Certificates.

Furthermore, the deletion of a Provisioning Certificate in Gireve's PCP with [Delete a PC in PCP](https://gireve-apis.stoplight.io/docs/pncp/2dze978qcx6ao-delete-a-pc-in-pcp) will deactivate the eventual SCCBs associated with that Provisioning Certificate in Gireve's CCP.

The PCID set as input parameter targets which PC to remove from Gireve PCP. Most of the time, a PCID targets exactly one PC. However, particularly in the case of a PC renewal, a PCID may target more than one active PC. For this reason, optional parameters `serial_number` (decimal format) and `issuer_dn` can also be set to ensure targeting exactly the right PC.

> **📝 Note:** If only the PCID is set as an input parameter and that PCID matches multiple valid PCs associated with that operator, all targeted PCs will be deleted from Gireve PCP.

### 3.8.4 Make an OEM Root available in Gireve RCP

<span id="MakeanOEM" class="anchor"></span>

If an OEM manages its own OEM Root Certificate, it must make it available in Gireve's RCP. This is necessary because the PCP and eMSP need this root certificate to validate the OEM's PCs. Without this information, these actors will reject the OEM's PC. This includes Gireve as a PCP and as an eMSP delegate.

For all these reasons, all OEMs connected to Gireve's Trust platform are required to make their root certificates available in Gireve's RCP. The process for making a Root Certificate available in Gireve's RCP is described in [Section 3.4.1](#making-a-root-certificate-available-in-the-gireve-rcp).

### 3.8.5 Be notified if SCCB linked to OEM's PC is added or removed from Gireve CCP

As detailed in [Section 3.5](#notification-services-guidelines), the OEMs can receive webhook notifications about the following events related to their EVs:

- `oem.contract.certificate.available`
- `oem.contract.certificate.revoked`

The first event indicates that an SCCB associated with a Provisioning Certificate for the OEM's EVs has been added to Gireve CCP and is ready for installation.

The second event notifies the OEM that the Contract Certificate linked to a Provisioning Certificate has been revoked. If this certificate has already been installed in an EV, the OEM must remove it.