# PNCP Implementation Guide

This repository contains a public, version-controlled implementation guide for the **Plug aNd Charge Protocol (PNCP)** used on Gireve's Trust Platform.

PNCP is an open protocol that defines how an operator's software platform interacts with a PKI ecosystem service provider to enable Plug & Charge services, including PKI services, V2G Root CA management, Certificate Provisioning Services and certificate pools.

It is intended to support the secure deployment of **ISO 15118 Plug & Charge** across the EV charging ecosystem.

The guide is primarily intended for technical teams — system administrators, developers, architects and project managers — integrating their systems with Gireve's Trust Platform via PNCP.

Readers should already be familiar with:

* PKI and digital certificates;
* the basic principles of ISO 15118-2.

Knowledge of OCPI is useful but not required.

---

## Documentation

Use the links below to go directly to the part of the documentation relevant to your implementation.

### 1. [Introduction](docs/01-introduction.md)

General introduction to Gireve Trust Platform and PNCP.

* [1.1 Aims](docs/01-introduction.md#11-aims)
* [1.2 Intended audience](docs/01-introduction.md#12-intended-audience)
* [1.3 Prerequisites](docs/01-introduction.md#13-prerequisites)
* [1.4 Document layout](docs/01-introduction.md#14-document-layout)
* [1.5 Definitions and abbreviations](docs/01-introduction.md#15-definitions-and-abbreviations)

---

### 2. [Technical implementation guidelines](docs/02-technical-guidelines.md)

Technical requirements and common principles that apply to all PNCP integrations.

* [2.1 PNCP principles](docs/02-technical-guidelines.md#21-pncp-principles)
* [2.2 PNCP Module pattern](docs/02-technical-guidelines.md#22-pncp-module-pattern)
* [2.3 URLs](docs/02-technical-guidelines.md#23-urls)
* [2.4 Concepts derived from the OCPI protocol](docs/02-technical-guidelines.md#24-concepts-derived-from-the-ocpi-protocol)

  * [X-Request-ID and X-Correlation-ID](docs/02-technical-guidelines.md#241-headers-x-request-id-and-x-correlation-id)
  * [Response format](docs/02-technical-guidelines.md#242-response-format)
  * [Status codes](docs/02-technical-guidelines.md#243-status-codes-returned)
  * [Pagination](docs/02-technical-guidelines.md#244-pagination)
* [2.5 Gireve Trust Platform environments](docs/02-technical-guidelines.md#25-gireve-trust-platform-environments)
* [2.6 Communication Partner and Operator](docs/02-technical-guidelines.md#26-communication-partner-and-operator)
* [2.7 Security](docs/02-technical-guidelines.md#27-security)

  * [Mutual TLS](docs/02-technical-guidelines.md#271-communication-encryption-and-client-authentication-via-mutual-tls)
  * [IP filtering](docs/02-technical-guidelines.md#272-ip-filtering)
  * [Web Application Firewall](docs/02-technical-guidelines.md#273-gireves-web-application-firewall)
  * [PKI security](docs/02-technical-guidelines.md#274-pki-security)
* [2.8 Client authentication and identification](docs/02-technical-guidelines.md#28-client-authentication-and-identification)

  * [Communication Partner authentication](docs/02-technical-guidelines.md#281-communication-partner-authentication)
  * [Operator identification via PNCP headers](docs/02-technical-guidelines.md#282-operator-identification-via-pncp-headers)

---

### 3. [Functional implementation guidelines](docs/03-functional-guidelines.md)

Functional workflows and PNCP services according to the role of each actor.

#### Common services

* [3.1 PNCP services by roles and actors](docs/03-functional-guidelines.md#31-pncp-services-by-roles-and-actors)
* [3.2 PNCP service permissions](docs/03-functional-guidelines.md#32-pncp-services-permissions-in-gireve-trust-platform)

#### PKI services

* [3.3 PKI services guidelines](docs/03-functional-guidelines.md#33-pki-services-guidelines)

  * [Certificate profiles](docs/03-functional-guidelines.md#331-certificates-profiles-definition-and-usage)
  * [Leaf certificate enrolment and associated SubCAs](docs/03-functional-guidelines.md#332-leaf-certificate-enrolment-and-obtention-of-linked-subcas)
  * [Certificate renewal](docs/03-functional-guidelines.md#333-renewal-of-a-leaf-certificate)
  * [Certificate revocation](docs/03-functional-guidelines.md#334-revocation-of-a-leaf-certificate)
  * [Certificate consultation](docs/03-functional-guidelines.md#335-certificate-consultation-services)

#### Root Certificate Pool (RCP)

* [3.4 RCP services guidelines](docs/03-functional-guidelines.md#34-rcp-services-guidelines)

  * [Make a Root Certificate available in the Gireve RCP](docs/03-functional-guidelines.md#341-making-a-root-certificate-available-in-the-gireve-rcp)
  * [Retrieve Root Certificates from the Gireve RCP](docs/03-functional-guidelines.md#342-retrieving-root-certificates-from-the-gireve-rcp)
  * [Root Certificate addition/removal notifications](docs/03-functional-guidelines.md#343-being-notified-that-a-root-certificate-has-been-added-or-removed-from-gireve-rcp)

#### Notifications and webhooks

* [3.5 Notification services guidelines](docs/03-functional-guidelines.md#35-notification-services-guidelines)

  * [How notifications work](docs/03-functional-guidelines.md#351-how-does-it-work)
  * [Webhook management](docs/03-functional-guidelines.md#352-webhook-management)
  * [Events triggering webhooks](docs/03-functional-guidelines.md#353-list-of-events-triggering-webhooks)
  * [Securing webhook endpoints](docs/03-functional-guidelines.md#354-how-to-secure-your-webhook-endpoints)

#### CPO implementation

* [3.6 CPO services guidelines](docs/03-functional-guidelines.md#36-cpo-services-guidelines)

  * [Install the necessary certificates in the Charging Point](docs/03-functional-guidelines.md#361-installation-of-all-necessary-certificates-in-the-charging-point)
  * [SECC certificate lifecycle management](docs/03-functional-guidelines.md#362-secc-certificate-lifecycle-management)
  * [Install the Contract Certificate in the vehicle](docs/03-functional-guidelines.md#363-install-the-contract-certificate-in-the-vehicle)
  * [Consult enrolled SECC Certificates](docs/03-functional-guidelines.md#364-consultation-of-enrolled-secc-certificate)
  * [Metadata usage for CPOs](docs/03-functional-guidelines.md#365-metadata-usage-for-cpos)
  * [Roaming authentication with eMAIds](docs/03-functional-guidelines.md#366-manage-roaming-authentication-workflow-with-emaids)

#### eMSP implementation

* [3.7 eMSP services guidelines](docs/03-functional-guidelines.md#37-emsp-services-guidelines)

  * [Create a Contract Certificate and make it available in the CCP](docs/03-functional-guidelines.md#371-creation-of-a-contract-certificate-and-making-it-available-in-ccp-for-installation-in-ev)
  * [Contract Certificate renewal](docs/03-functional-guidelines.md#372-contract-certificate-renewal)
  * [Consult enrolled Contract Certificates](docs/03-functional-guidelines.md#373-consultation-of-enrolled-contract-certificate)
  * [Contract Certificate expiration notification](docs/03-functional-guidelines.md#374-contract-certificate-expiration-notification)
  * [Contract Certificate revocation and removal from CCP](docs/03-functional-guidelines.md#375-contract-certificate-revocation-and-removal-from-ccp)
  * [Make the eMSP Root Certificate available in an RCP](docs/03-functional-guidelines.md#376-making-the-emsp-root-certificate-available-in-an-rcp)
  * [Retrieve an OEM PC from a PCID](docs/03-functional-guidelines.md#377-retrieve-an-oem-pc-from-a-pcid)
  * [Check an OEM PC from a PCID](docs/03-functional-guidelines.md#378-check-an-oem-pc-from-a-pcid)
  * [Retrieve an OEM Root Certificate from Gireve RCP](docs/03-functional-guidelines.md#379-retrieve-oem-root-certificate-in-gireve-rcp)
  * [SCCB removal/retrieval notifications](docs/03-functional-guidelines.md#3710-be-notified-if-sccb-is-removed-or-retrieved-from-gireve-ccp)
  * [Roaming authentication with eMAIds](docs/03-functional-guidelines.md#3711-manage-roaming-authentication-workflow-with-emaids)

#### OEM implementation

* [3.8 OEM services guidelines](docs/03-functional-guidelines.md#38-oem-services-guidelines)

  * [Install the necessary certificates in the EV](docs/03-functional-guidelines.md#381-installation-of-all-necessary-certificates-in-the-ev)
  * [Provisioning Certificate lifecycle management](docs/03-functional-guidelines.md#382-pc-certificate-lifecycle-management)
  * [Provision and administer PCs in Gireve pools](docs/03-functional-guidelines.md#383-provision-and-administration-of-pcs-in-gireve-pools)
  * [Make an OEM Root available in Gireve RCP](docs/03-functional-guidelines.md#384-make-an-oem-root-available-in-gireve-rcp)
  * [SCCB notifications for OEM Provisioning Certificates](docs/03-functional-guidelines.md#385-be-notified-if-sccb-linked-to-oems-pc-is-added-or-removed-from-gireve-ccp)

---

### 4. [PKI and digital certificates](docs/04-pki-and-certificates.md)

Background information for readers who need an introduction or refresher on PKI and X.509 certificates.

* [What is a digital certificate?](docs/04-pki-and-certificates.md#411-what-is-a-digital-certificate)
* [How are certificates created?](docs/04-pki-and-certificates.md#412-how-do-we-make-them)
* [Certificate chains](docs/04-pki-and-certificates.md#413-the-concept-of-certificate-chain)
* [Certificate chain validation](docs/04-pki-and-certificates.md#414-explanation-about-certificate-chain-validation)
* [Certificate revocation](docs/04-pki-and-certificates.md#415-about-certificate-revocation)
* [Certificate renewal](docs/04-pki-and-certificates.md#416-about-certificate-renewal)

---

### 5. [ISO 15118-2 and Plug & Charge](docs/05-iso15118-plug-and-charge.md)

Background information on ISO 15118 and the Plug & Charge ecosystem.

* [5.1 Global overview](docs/05-iso15118-plug-and-charge.md#51-global-overview)
* [5.2 Roles and actors](docs/05-iso15118-plug-and-charge.md#52-roles-and-actors)
* [5.3 Main use cases](docs/05-iso15118-plug-and-charge.md#53-main-use-cases)

  * [OEM produces an ISO 15118-ready vehicle](docs/05-iso15118-plug-and-charge.md#531-the-oemcar-maker-produces-an-iso-15118-ready-vehicle)
  * [EV User subscribes to an eMSP](docs/05-iso15118-plug-and-charge.md#532-the-ev-user-subscribes-to-an-emsp)
  * [EV User activates Plug & Charge](docs/05-iso15118-plug-and-charge.md#533-the-ev-user-activates-the-plugcharge-feature-in-its-emsp-subscription-for-its-vehicle)
  * [OEM installs the Contract Certificate in the vehicle](docs/05-iso15118-plug-and-charge.md#534-the-oemcar-maker-installs-the-contract-certificate-in-the-vehicle)
  * [Vehicle connects to an ISO 15118-ready Charging Point](docs/05-iso15118-plug-and-charge.md#535-the-ev-user-plugs-its-iso-15118-ready-vehicle-on-an-iso-15118-ready-charging-point)
  * [CPO prepares an ISO 15118-ready Charging Station](docs/05-iso15118-plug-and-charge.md#536-the-cpo-prepares-an-iso-15118-ready-charging-station)



Version 1.2.0 adds support for enrolled certificate consultation, metadata search and certificate expiry notifications compared with previous versions.

Future changes to PNCP or Gireve Trust Platform will be reflected in this repository through standard Git versioning and tagged releases.
