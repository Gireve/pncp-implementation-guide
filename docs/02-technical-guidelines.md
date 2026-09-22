# 2 Technical implementation guidelines

## 2.1 PNCP principles

PNCP protocol is based on the same principles as OCPI protocol, to reduce the complexity for actors using OCPI for roaming (like CPOs and eMSPs).

The following basic principles are inherited from this "proximity" with OCPI.

- GIREVE Trust Platform APIs use Json/Rest standards.
- GIREVE Trust Platform APIs are structured in "modules" that group APIs on a functional basis.

But some OCPI mechanisms are not relevant for the PNCP features and thus have not been integrated in the protocol:

- There is no credential-like module in PNCP.
- The authentication is done through OAuth2 mechanism.

> [!NOTE]
>  For security reasons, the GIREVE Roaming platform and the GIREVE Trust Platform do not share a unique partner authentication mechanism nor a common credential management.

## 2.2 PNCP Module pattern

To fulfil the principle of "proximity" with OCPI, GIREVE has implemented 4 custom modules[^1]:

- **PNCGenericCertificate module**: Module used to manage x509 certificates.
- **PNCRootCACertificate module**: Module used to manage ISO-15118-Plug&Charge Root-CA Certificates.
- **PNCProvisioningCertificate module**: Module used to manage ISO-15118-Plug&Charge Provisioning Certificates of EVs.
- **PNCContractCertificate module**: Module used to manage ISO-15118-Plug&Charge contract certificates.

Custom modules defined by GIREVE for PNCP, following the OCPI module pattern.

## 2.3 URLs

The PNCP URLs are structured by the following pattern:

- URL = `https://<environment>.gireve.com/pncp/<API_Version>/<module_name>`

- with
  - `<environment>` which describes the production or preproduction environment.
    - Production environment: `"trust-secure-api"`
    - Preproduction environment: `"pp-trust-secure-api"`
  - `<API_Version>` which designate the version of the protocol.
    - Structure following the pattern `"x.y.z"`
  - `<module_name>` which designates the module:
    - `"PNCContractCertificate"`
    - `"PNCProvisioningCertificate"`
    - `"PNCRootCACertificate"`
    - `"PNCGenericCertificate"`

- Example
  - `https://trust-secure-api.gireve.com/pncp/1.0.2/PNCContractCertificate`
  - is a valid URL for
    - `"PNCContractCertificate"` module
    - Version `"1.0.2"`
    - On production environment

## 2.4 Concepts derived from the OCPI protocol

### 2.4.1 Headers "X-Request-ID" and "X-Correlation-ID"

Used for tracing and debugging, the headers *"X-Request-ID"* and *"X-Correlation-ID"* introduced in OCPI 2.2 are used in GIREVE Trust Platform APIs.

*Please see <https://github.com/ocpi/ocpi/blob/2.2.1/transport_and_format.asciidoc#12-unique-message-ids> (from OCPI standard description on Github).*

> [!IMPORTANT]
> The implementation of the headers *"X-Request-ID"* and *"X-Correlation-ID"* are mandatory.

### 2.4.2 Response format

Responses returned by GIREVE Trust Platform services are compliant with the standard OCPI response format.

*Please see <https://github.com/ocpi/ocpi/blob/2.2.1/transport_and_format.asciidoc#117-response-format> (from OCPI standard description on Github).*

### 2.4.3 Status codes returned

Status codes returned by GIREVE Trust Platform APIs are compliant with the OCPI standard.

*Please see <https://github.com/ocpi/ocpi/blob/2.2.1/transport_and_format.asciidoc#117-response-format> (from OCPI standard description on Github).*

### 2.4.4 Pagination

Some PNCP services, such as GetRootCA, reuse OCPI paging principles by using offset and limit parameters in the request as well as the "link", "X-Total-Count" and "X-limit" headers in the response.

*Please see <https://github.com/ocpi/ocpi/blob/2.2.1/transport_and_format.asciidoc#pagination> (from OCPI standard description on Github).*

## 2.5 Gireve Trust Platform environments

Two main environments are available:

- the pre-production environment;
- the production environment.
  
All PNCP connections to these environments require:

- HTTPS with TLS 1.3;
- a client TLS certificate with the private key corresponding to that certificate;
- valid OAuth 2.0 access token generate with the client OAuth 2.0 credentials .

The client TLS client certificate and OAuth 2.0 credentials are provided during the technical onboarding process.

### 2.5.1 Preproduction environment

This environment enables partners to test, refine and validate their implementation before going live. It can also be used to create proof-of-concept tools or demonstrations.

All Root-CAs, Sub-CAs and leaf certificates managed in the Preproduction environment can only be used for test or demonstration purposes. They cannot be used in a production context.

The base URL for the mTLS-secured pre-production environment is : 

<https://pp-trust-secure-api.gireve.com>

### 2.5.2 Production environment

This is the environment for service delivery.

The base URL for the mTLS-secured pre-production environment is :

<https://trust-secure-api.gireve.com/...>

## 2.6 Communication partner and operator

![Communication partner and operator diagram](./images/media/image2.png)

### 2.6.1 Communication partner

A Communication Partner is a system actor that is technically connected to the Gireve Trust Platform and exchanges messages, including requests and responses, with it.

The Communication Partner acts as a client in the client-server communication.

A Communication Partner may host one or several Operators, including CPOs, eMSPs, OEMs, CCPs or PCPs.

Authentication and identification of a communication partner are detailed in section [Section 2.8](#28-client-authentication-and-identification).

**Example:** A CPO system or an eMSP "back-end" system, connected to the GIREVE's Platform, is a Communication Partner.

### 2.6.2 Operator

An operator is the actor in the system who is functionally connected to the GIREVE platform and exchanges messages with it. This is a business role, and the company to which it is linked has a "Platform Access Contract" with GIREVE.

Operators can have one or more communication partners for the management and monitoring of their data flows.

The separation between the two roles of "Operator" and "Communication Partner" is particularly important when a Communication Partner hosts several Operators. This notion also makes it possible to distinguish all the Communication Partners of an Operator.

**Example:** a CPO or an eMSP is an Operator. A CPO system is a Communication Partner of the CPO.

Some actors use the term "Mandant" to designate functional actor using a given system. In this case "Mandant" is a synonym of "Operator". Some actors use the terms of "Sub-Operator" (Sub-CPO, Sub-eMSP, Sub-OEM ...) to designate functional actor using a given system. In this case "Sub-Operator" is a synonym of "Operator".

## 2.7 Security

The security of the Gireve Trust Platform relies on complementary mechanisms implemented at different levels:

- TLS 1.3 for encryption and integrity of communications;
- mutual TLS for authentication of the connecting technical client;
- OAuth 2.0 for API client authentication and authorisation;
- IP address filtering;
- web application firewall protection;
- secure storage and use of certification authority private keys.

### 2.7.1 Communication encryption and client authentication via mutual TLS

Communication between the client system and the Gireve Trust Platform is secured and encrypted at the transport layer using TLS 1.3.

TLS, or Transport Layer Security, establishes a secure and encrypted connection between the client and the server. It protects the communication against eavesdropping and data tampering.

PNCP API connections use mutual TLS authentication, also referred to as mTLS.

During the TLS handshake:
- the Gireve Trust Platform presents its server certificate;
- the client verifies the server certificate and its certificate chain;
- the client presents its client TLS certificate;
- the Gireve Trust Platform verifies the client certificate and confirms that it is authorised for the relevant environment.
- The verification of the server certificate enables the client to confirm that it is communicating with the legitimate Gireve Trust Platform endpoint.

The verification of the client certificate enables the Gireve Trust Platform to authenticate the technical system establishing the connection.

After successful authentication and key exchange, the client and server establish the encryption keys used to protect the data exchanged during the TLS session. This ensures the confidentiality and integrity of the information exchanged.


> [!NOTE]
>  Gireve Trust platform presents a server certificate linked to the Google's Root Certificate Authority **"GTS Root R1"** which can be downloaded here: <https://pki.goog/repository/>.
> The calling partner system needs to recognize this Root Certificate Authority as a trusted entity to allow the TLS communication.

**Note:** "GTS Root R1" is already included in most recent trust store (like for java in the cacerts file of the JVM's security module).

**Note 2:** The client must present a valid client TLS certificate during the TLS handshake. This client certificate and a valid OAuth 2.0 access token are necessary for the client authentication to the platform as described in [Section 2.8](#28-client-authentication-and-identification).


### 2.7.2 IP Filtering

Calling partners must provide a list of authorised IP addresses before connecting to the platform.

The Gireve Trust platform also uses customer IP filtering as an additional security measure. This ensures that only trusted sources can access the API, reinforcing security and protecting against unauthorized access attempts.

### 2.7.3 Gireve's web application firewall

Gireve's Trust platforms are protected by a powerful web application firewall (WAF). This firewall features anti-DDoS protection and includes numerous and complex protection rules to prevent any intrusion attempts into the system and guarantee a maximum level of security.


> [!NOTE]
> One of the rules in place to protect against CRLF injections will rejects request with a JSON body not "minified". This means containing white spaces or line breaks between the JSON attributes.
> If you make test calls via Postman on the pre-production platform you need to "minify" your JSON payload, otherwise they'll be rejected by the platform.

### 2.7.4 PKI security

The storage and use of the certificate authority private keys is secured in some Hardware Secure Modules (HSM). Hardware security modules act as trust anchors that protect the cryptographic infrastructure by securely managing, processing, and storing cryptographic keys inside a hardened, tamper-resistant device.

## 2.8 Client authentication and identification

Access to the Gireve Trust Platform relies on complementary authentication and identification mechanisms:

1. the Communication Partner's technical system is authenticated using mutual TLS and a client TLS certificate;
2. the API client is authenticated and authorised using an OAuth 2.0 access token;
3. the source Operator is identified using mandatory PNCP headers;
4. the connection source is checked against the authorised IP addresses.

All applicable authentication, identification and security checks must succeed for the request to be accepted and processed.


### 2.8.1 Communication Partner authentication

The Communication Partner is authenticated using two complementary mechanisms:

mutual TLS authentication at the transport layer;
OAuth 2.0 authentication and authorisation at the application layer.
A successful mutual TLS connection does not remove the requirement to provide a valid OAuth 2.0 access token.

Likewise, a valid OAuth 2.0 access token does not remove the requirement to present a valid client TLS certificate during the TLS handshake.

Client authentication and identification on our platform is based on OAuth2 tokens, a technology recognized for its reliability in securing access. These tokens are generated by the client from information previously transmitted to the connection by Gireve, during the technical connexion stage, notably the **client_id** and **client_secret**.


#### Mutual TLS authentication
The Communication Partner must present a valid client TLS certificate when establishing a connection to the Gireve Trust Platform.

The client TLS certificate:

- authenticates the technical system establishing the TLS connection;
- must have been provisioned or approved by Gireve during the technical onboarding process;
- must be accepted for the target environment;
- must correspond to the private key used by the connecting system;
- must be presented to the Gireve Trust Platform during the TLS handshake.
- The client TLS certificate is generated and provisioned as part of the technical onboarding process conducted with the Gireve Connection team.

The certificate and its associated private key must be installed on the technical component that establishes the TLS connection with the Gireve Trust Platform.

If a proxy, API gateway or other intermediary terminates the TLS connection, the client TLS certificate must be configured on that component.

The private key associated with the client TLS certificate must remain under the exclusive control of the Communication Partner and must never be transmitted to Gireve.

Successful mutual TLS authentication establishes and authenticates the secure technical connection. An OAuth 2.0 access token remains mandatory to access the PNCP APIs.

> [!IMPORTANT]  
> The private key associated with the client TLS certificate must remain under the exclusive control of the Communication Partner and must not be transmitted to Gireve.

> [!WARNING]
> **Legacy integrations:** Communication Partners onboarded before the introduction of mutual TLS may temporarily continue to authenticate using only their OAuth 2.0 Token. This transitional arrangement applies only to existing integrations. All new PNCP integrations must use mutual TLS in addition to OAuth 2.0.


#### OAuth 2.0 authentication
API authentication and authorisation use OAuth 2.0 access tokens.

The OAuth 2.0 access token is generated by the Communication Partner using the credentials provided by Gireve during the technical onboarding process, including the `client_id` and `client_secret`.

The Communication Partner must generate a valid OAuth 2.0 access token using the credentials provided by Gireve.

If the access token is missing, invalid or expired, the API request is rejected.

The generated access token must be included in the Authorization HTTP header using the following format:

`Authorization: Bearer <access_token>`

> [!IMPORTANT]  
> OAuth2 tokens have a limited validity period (a few hours). **The calling partner must reuse an existing token as long as it remains valid** and only request a new token upon expiration. This technical requirement would be verified during the certification process. The access token must be renewed when it reaches the end of its validity period.



### 2.8.2 Operator identification via PNCP headers

As explained in [Section 2.6](#26-communication-partner-and-operator), when a partner system calls us, it does so on behalf of an operator who represents the functional actor (eMSP, CPO, OEM, CCP, PCP).

The operator, source of the request, is identified using the **PNCP-from-country-code** and **PNCP-from-party-id** headers, which are populated respectively by the operator's country code and party id, as defined by the eMI3 group standard.

> [!IMPORTANT]  
> For each call, calling partner systems must fill in the mandatory headers `PNCP-from-country-code` and `PNCP-from-party-id`. Gireve will then check that the Operator has been declared in its system and that the Communication-Partner is attached to this Operator. If either of these conditions is not met, the call will be rejected.

**Note:** If the Operator doesn't have the "country code" and "party id" information, Gireve can create them for him.

[^1]: The names of these modules have been prefixed by “PNC” (for “Plug’n Charge”) to avoid any confusion in case of OCPI defines an OCPI standard plug & charge related module and/or object.
