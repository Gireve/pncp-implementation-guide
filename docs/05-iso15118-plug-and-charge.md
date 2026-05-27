## 5 APPENDIX 2: ISO-15118-2 and Plug Charge environment

### 5.1 Global overview

#### 5.1.1 ISO-15118 and the data link between electric vehicle and charging station

*"ISO-15118 Road vehicles -- Vehicle to grid communication interface" is an international standard defining a vehicle to grid (V2G) communication interface for bi-directional charging/discharging of electric vehicles. The standard provides a Plug Charge feature used by some electric vehicle networks.[^1]*

The standard describes a secure connection between electric vehicles and charging stations. This V2G-Datalink has been designed to provide the messages and data exchanges needed to implement several use cases, including "Charging Process", "Authentication and authorization", and "Smart-charging and V2G".

![ISO-15118 overview](./images/media/image18.png)

This document is focused on the establishment of the "V2G-Datalink" and the "Plug&Charge authentication and authorization" use-cases.
There are two versions of ISO-15118: the "-2" released in 2014 and the "-20" released in 2022. This document is based on the "-2" version but the main differences of "-20" are described and highlighted.

The "V2G-Datalink" uses "TLS"[^2] protocol to establish a secure IT communication. This connection is secured by an authentication of the parties involved, based on X509 certificates:

- The vehicle authenticates the charging-station: The charging-station presents its certificate, and the vehicle verifies it to ensure it can trust the charging station. This certificate is named "**Charging Station Certificate**", or "**Charging Point Certificate**", or "**SECC Certificate**".
- In ISO-15118-20 only, the authentication is mutual, and the charging station also authenticates the vehicle: The vehicle presents its certificate, and the charging-station verifies it and checks whether it can trust the vehicle. Please note that, for this feature, the vehicle presents a specific x509 certificate which is not the "**Provisioning Certificate**" but a dedicated one. In this document, we will refer to this certificate as the "**Vehicle certificate**".

If this TLS connection fails, for reasons related to the certificate or for any other reason, no ISO-15118 functions can be activated, and use cases will stop. The Plug Charge system will not work.

[^1]: Wikipedia — [https://en.wikipedia.org/wiki/ISO_15118](https://en.wikipedia.org/wiki/ISO_15118)
[^2]: TLS stands for Transport Layer Security, which is a cryptographic protocol designed to provide communications security over a computer network. [Wikipedia](https://en.wikipedia.org/wiki/Transport_Layer_Security)

#### 5.1.2 Plug Charge feature

The Plug Charge system is a solution in which EV-Users authentication is automated and initiated by the vehicle and therefore does no longer rely on a RFID badge, or smartphone app etc. From the user point of view, it simplifies the process: *"I plug, it charges!"*.

Plug Charge is based on two principles:

- The user subscribes to an eMobility Service Provider (eMSP) which will pay the charging session to the Charging Point Operator (CPO). Authenticating the user ultimately means authenticating the relation between the user and his eMSP. This relation is identified by an identifier called the eMobility Account Identifier (eMAId).
- The vehicle has a X509 certificate that authenticates the relation between the user and its eMSP. This certificate is called the Contract Certificate. When the vehicle is plugged to the charging station and after the V2G-Datalink has been established, it presents this Contract Certificate to the Charging Station, and its CPO will:
  - **Authenticate** the eMSP-user contract, by verifying the Contract Certificate and ensuring that it is trustworthy.
  - Perform the **authorization** process, based on the eMAId contained in the certificate.

Please note that there are two different steps: the **authentication** is a process based on cryptography and PKI, and **authorization** is an e-mobility process. 
The charging session will only start if both authentication and authorization are successful.

#### 5.1.3 ISO-15118 and Plug Charge ecosystem

In previous chapters, we saw that ISO-15118 and Plug Charge require 3 certificates:

- Charging Station Certificate
- Vehicle Certificate (only for ISO-15118-20)
- Contract Certificate

The generation and installation of the Charging Station Certificate and Vehicle Certificate are conventional: the target generates its key pair, a PKI issues the certificate, and the certificate is stored in the target during the installation process.

The generation of the Contract Certificate is more complex as the initiative and generation of the key pair are not managed by the target (which is the vehicle), but by the user and his eMSP. To perform this generation and the necessary transfer to the vehicle in which the certificate must ultimately be installed, authenticated data related to the vehicle is required. These are stored in the vehicle Provisioning Certificate.

The principal steps in this process are as follows:

- The OEM generates the "Provisioning Certificate" of each vehicle.
- A transmission mechanism, named "Provisioning Certificate Pool", makes the "Provisioning Certificate" available for the eMSP.
- The eMSP generates the "Contract Certificate" and packages it in a Signed "Contract Certificate Bundle".
- A transmission mechanism, named "Contract Certificate Pool", makes the "Contract Certificate" available to the CPO or OEM for its installation in the vehicle using:
  - the "CPO → Charging Station → Vehicle" path. Or
  - the "OEM-back-end → Vehicle" path.

![ISO-15118 and Plug Charge ecosystem](./images/media/image19.png)

All these actors, transmission means, and processes form the "ISO-15118 and Plug Charge ecosystem".

#### 5.1.4 ISO-15118 Certificates — Summary

The following table summarizes the main characteristics of the certificates involved in ISO-15118 and Plug&Charge.

![ISO-15118 Certificates Summary table](./images/media/image20.png)

*Table 1 — ISO-15118 Certificates — Summary*

### 5.2 Roles and actors

![Roles and actors diagram](./images/media/image21.png)

The main actor roles involved are:

- **The vehicle**
  - Contains the Provisioning Certificate, the Vehicle certificate (ISO-15118-20) and the Contract Certificate.
  - Receives a Signed Contract Certificate Bundle and registers the Contract Certificate it contains.
  - Authenticates the SECC Certificate attached to a list of recognized V2G Root Certificate Authorities.

- **The EV-User**
  - Uses/has a vehicle compatible with PlugCharge.
  - Subscribes to an eMSP.
  - The PlugCharge feature is included in his eMSP subscription for his vehicles.

- **The OEM/Car-Maker**
  - Produces ISO-15118-ready vehicles.
  - Installs the Provisioning certificate and the Vehicle Certificates (ISO-15118-20) in the vehicle.
  - Makes the Provisioning Certificates available in a Provisioning Certificate Pool.
  - Option: Retrieve the eMSP contract certificate packed in a SCCB from a CCP and transmit it to the vehicle for installation.

- **The OEM-Side PKI**
  - Generates the vehicle Provisioning Certificates.
  - Manages the PCs lifecycle (revocation and renewal).
  - The Root-CA linked to the PC is not necessarily a V2GRoot-CA.

- **The eMSP**
  - Creates the contract certificate bundle (CCB) containing the CC and other necessary information for its future installation in the vehicle.
  - Packages the Contract Certificate in a Signed Contract Certificate Bundle.
  - Has their bundles signed by a CPS.
  - Makes the SCCBs available in a CCP for their future installation in the vehicle.
  - Authorizes the charge.

- **The eMSP-Side PKI**
  - Generates the Contract Certificates.
  - Manages the CCs lifecycle (revocation and renewal).
  - The Root-CA linked to the CC is not necessarily a V2GRoot-CA.

- **The CPO**
  - Installs a certificate (SECC) issued from a V2G PKI in each of its charging stations, so that the vehicle can securely identify them.
  - Retrieves the eMSP contract certificate packed in a SCCB from a CCP and transmits it to the vehicle for installation.

- **The CPO-side PKI and its V2G-RootCA anchor**
  - Generates the SECC Certificates.
  - Manages the SECC lifecycle (revocation and renewal).
  - The Root-CA linked to the SECC **must be** a V2GRoot-CA.

- **The Provisioning Certificates Pool (PCP)**
  - Stores and makes available the PC of the OEM manufacturers.
  - Must be able to retrieve a PC from another PCP if they don't have it in their own pool (in a context of pool interoperability).

- **The Contract Certificates Pool (CCP)**
  - Stores and makes available the valid SCCBs (not linked to an expired or revoked CC or PC).
  - CPOs and OEMs can come and get them.
  - It is the responsibility of the CCP to make the necessary security checks (check the request, check validity on the PC in the request, etc.) when it receives the certificate installation requests.
  - Must be able to retrieve a SCCB from another CCP if they don't have it in their own pool (in a context of pool interoperability).

- **The CPS and its V2G-RootCA anchor**
  - Must have some CPS leaf certificates issued from V2G PKIs to be able to sign the SCCB.
  - Before signing the contract certificate bundle, it must verify the validity of the CC provided.
  - Signs the CCB (it becomes the SCCB) with the private key linked to a selected CPS leaf certificate.

### 5.3 Main Use-cases

The main use cases are as follows:

1. The OEM/Car-maker produces an ISO-15118-ready vehicle
2. The EV-User subscribes to an eMSP
3. The EV-User activates the PlugCharge feature in its eMSP subscription, for its vehicle
4. (option) The OEM/Car-maker installs the Contract certificate in the vehicle
5. The EV-User plugs its ISO-15118-ready vehicle on an ISO-15118-ready charging point
6. The CPO prepares an ISO-15118-ready Charging Station

![Main use-cases overview](./images/media/image22.png)

#### 5.3.1 The OEM/Car-maker produces an ISO-15118-ready vehicle

An ISO-15118 Plug Charge ready vehicle must have some specific hardware and software components. These requirements and actions are entirely internal to the car maker. The specific actions involving the fact that the vehicle must comply with ISO-15118 and Plug Charge, on which we will focus in this document, are related to the PKI and Plug Charge ecosystem:

- Install the relevant V2G root-certificates in the vehicle.
- Generate the provisioning certificate of the vehicle and make it available

![OEM produces ISO-15118-ready vehicle](./images/media/image23.png)
*Figure 5 - The OEM/Car-maker produces an ISO-15118-ready vehicle*

The car-maker backend system:

- Must generate the Provisioning Certificate Identifier (PCiD) in compliance with the standard definition (see step 1.0 on illustration).
- Must transmit to the vehicle the relevant V2G root-CA certificates, to be installed in it (see step 1.1). This could be done using a "V2G-Root-CA-TrustList" or individually, certificate per certificate.

The vehicle:

- Must receive and install the relevant V2G root-CA certificates in its "Trust store" to ensure a valid authentication of the trusted charging points, and of the trusted "Signed contract certificate bundles" (see step 1.1).
- Must generate a key-pair, store it in its "key-store" (see step 1.2) and request a certificate generation based on this pair: the Provisioning Certificate. This certificate "CN" field must contain the PCiD (see step 1.3).

The vehicle and the car-maker backend system:

- Must make available the Provisioning Certificate to all eMSPs, using a Provisioning Certificate Pool (see step 1.4).

#### 5.3.2 The EV-User subscribes to an eMSP

This step is not very impacted by the ISO-15118 and PlugCharge feature. There is no specific action related to these features. But this step is really important and mandatory because it generates the eMSP EV-user contract and its identifier eMAId (see 2.0 in the following illustration).
- Generate the eMAId.
![EV-User subscribes to an eMSP](./images/media/image24.png)
*Figure 6 - The EV-User subscribes to an eMSP*

The eMSP backend system:

- Must generate the eMAId (see step 2.0).

#### 5.3.3 The EV-User activates the PlugCharge feature in its eMSP subscription, for its vehicle

For the Plug&Charge feature, the authentication verified for each session is based on a x509 certificate named "Contract Certificate", which contains the eMAId identifier. The activation process must:

- Generate the contract certificate and the required associated data elements.
- Package them in a "contract certificate bundle".
- Sign this bundle with a CPS entity attached to a V2G-RootCA.
- Make the signed bundle available.

![EV-User activates PlugCharge](./images/media/image25.png)
*Figure 7 - The EV-User activates the Plug&Charge feature in its eMSP subscription, for its vehicle.*

The user asks his eMSP to activate the PlugCharge function for a given vehicle:

- The user sends his vehicle "PCiD" to his eMSP (see step 3.0).

The eMSP system:

- Must retrieve the vehicle Provisioning Certificate knowing the PCiD (see step 3.1).
- Must generate the Contract Certificate (see step 3.2).
- Must package the Contract certificate with additional data in a contract certificate bundle.
- Must have this bundle signed by a CPS attached to a V2G-RootCA (see step 3.3).
- Must make available the "Signed Contract Certificate Bundle" to the Car-Maker and to all CPOs by depositing it in a Contract Certificate Pool (see step 3.4).

#### 5.3.4 The OEM/Car-maker installs the Contract certificate in the vehicle

The contract certificate must be transferred to the vehicle for installation. This can be done via the OEM's back-end system, or via the charging point.
We describe below the situation in which the Contract Certificate is transferred via the OEM's back-end system.

![OEM installs contract certificate in vehicle](./images/media/image26.png)
*Figure 8 - The OEM/Car-maker installs the Contract certificate in the vehicle*

The OEM's backend system:

- Must get the Signed Contract Certificate Bundle from the Contract certificate Pool (see step 4.0).
- Must send the Signed Contract Certificate Bundle to the vehicle, using the telematic data link to the vehicle (see step 4.1).

The vehicle:

- Must receive the Signed Contract Certificate Bundle, unwrap, verify, and install it. The vehicle now contains the Contract Certificate.

#### 5.3.5 The EV-User plugs its ISO-15118-ready vehicle on an ISO-15118-ready charging point

When the EV-User plugs its vehicle on an ISO-15118-ready charging point, two situations could be encountered:

1. The vehicle presents a valid Contract Certificate to the Charging Point.
2. The vehicle has no valid Contract Certificate to be presented to the Charging Point.

If the vehicle presents a valid Contract Certificate to the Charging Point, the authentication process, and then the authorization process will be performed, and the charging session will start if the authorization is OK.

If the vehicle has no valid Contract Certificate to be presented to the Charging Point, the Charging Point will try to get one and to transfer it to the vehicle.

![EV-User plugs vehicle on charging point](./images/media/image27.png)
*Figure 9 - The EV-User plugs its ISO-15118-ready vehicle on an ISO-15118-ready charging point*

The vehicle:

- Is plugged to the Charging Point and the ISO-15118 data link is established (see step 5.0).
- Has no Contract Certificate to be presented to the Charging Station (see step 5.1), and thus requests the Charging Point for a Contract Certificate if available.

The Charging Station:

- Requests its CPO backend system (see step 5.2).

The CPO backend system:

- Retrieves the available "Contract Certificates Bundle" (if any) for this vehicle/PCiD (see step 5.3).
- Sends it back to the Charging Station (see step 5.4).

The Charging Station:

- Pushes the "Contract Certificate Bundle" to the vehicle (see step 5.5).

The vehicle:

- Must receive the "Contract Certificate Bundle", unwrap, verify, and install it. The vehicle now contains the "Contract Certificate". At this step, the vehicle can present a valid "Contract Certificate" to the Charging Point. The authentication process, and then the authorization process can be performed, and the charging session can start if the authorization is OK.

#### 5.3.6 The CPO prepares an ISO-15118-ready Charging Station

A Charging Station compliant with ISO-15118 must have a Certificate for each SECC. The usual implementations are one SECC per Charging Point or one SECC per Charging Station. Whatever this topology, each SECC must have a Certificate. The CPO must:

- Generate the SECC certificate.
- Install the certificate in the SECC.

![CPO prepares ISO-15118-ready Charging Station](./images/media/image28.png)

The CPO backend system:

- Must generate the SECC Identifier (SECCId) in compliance with the standard definition (see step 6.0 on illustration).

The SECC:

- Must generate a key-pair, store it in its key-store (see step 6.1) and request a certificate generation based on this pair: the SECC Certificate. This certificate CN field must contain the SECCId (see step 6.2).