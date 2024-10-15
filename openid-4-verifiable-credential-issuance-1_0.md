%%%
title = "OpenID for Verifiable Credential Issuance - Editor's draft"
abbrev = "openid-4-verifiable-credential-issuance"
ipr = "none"
workgroup = "OpenID Digital Credentials Protocols"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-4-verifiable-credential-issuance-1_0-15"
status = "standard"

[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="SPRIND"
    [author.address]
    email = "torsten@lodderstedt.net"

[[author]]
initials="K."
surname="Yasuda"
fullname="Kristina Yasuda"
organization="SPRIND"
    [author.address]
    email = "kristina.yasuda@sprind.org"

[[author]]
initials="T."
surname="Looker"
fullname="Tobias Looker"
organization="Mattr"
    [author.address]
    email = "tobias.looker@mattr.global"

%%%

.# Abstract

This specification defines an API for the issuance of Verifiable Credentials.

{mainmatter}

# Introduction

This specification defines an OAuth-protected API for the issuance of Verifiable Credentials. Credentials can be of any format, including, but not limited to, IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mDL [@ISO.18013-5], and W3C VCDM [@VC_DATA].

Verifiable Credentials are very similar to identity assertions, like ID Tokens in OpenID Connect [@OpenID.Core], in that they allow a Credential Issuer to assert End-User claims. A Verifiable Credential follows a pre-defined schema (the Credential type) and MAY be bound to a certain holder, e.g., through Cryptographic Holder Binding. Verifiable Credentials can be securely presented for the End-User to the RP, without involvement of the Credential Issuer.

Access to this API is authorized using OAuth 2.0 [@!RFC6749], i.e., the Wallet uses OAuth 2.0 to obtain authorization to receive Verifiable Credentials. This way the issuance process can benefit from the proven security, simplicity, and flexibility of OAuth 2.0 and existing OAuth 2.0 deployments and OpenID Connect OPs (see [@OpenID.Core]) can be extended to become Credential Issuers.

## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all capitals, as shown here.

# Terminology

This specification uses the terms "Access Token", "Authorization Endpoint", "Authorization Request", "Authorization Response", "Authorization Code Grant", "Authorization Server", "Client", "Client Authentication", "Client Identifier", "Grant Type", "Refresh Token", "Token Endpoint", "Token Request" and "Token Response" defined by OAuth 2.0 [@!RFC6749], the terms "End-User", "Entity", and "Request Object" as defined by OpenID Connect Core [@!OpenID.Core], the term "JSON Web Token (JWT)" defined by JSON Web Token (JWT) [@!RFC7519], the term "JOSE Header" defined by JSON Web Signature (JWS) [@!RFC7515].

Base64url-encoded denotes the URL-safe base64 encoding without padding defined in Section 2 of [@!RFC7515].

This specification also defines the following terms. In the case where a term has a definition that differs, the definition below is authoritative for this specification.

Credential Dataset:
:  A set of one or more claims about a subject, provided by a Credential Issuer.

Credential (or Verifiable Credential):
:  An instance of a Credential Configuration with a particular Credential Dataset, that is signed by an Issuer and can be cryptographically verified. An Issuer may provide multiple Credentials as separate instances of the same Credential Configuration and Credential Dataset but with different cryptographic values. In this specification, the term "Verifiable Credential" is also referred to as "Credential". It's important to note that the use of the term "Credential" here differs from its usage in [@!OpenID.Core] and [@!RFC6749]. In this context, "Credential" specifically does not encompass other meanings such as passwords used for login credentials..

Credential Format:
:  Data Model used to create and represent Credential information. This format defines how various pieces of data within a Verifiable Credential are organized and encoded, ensuring that the Verifiable Credential can be consistently understood, processed, and verified by different systems. The exact parameters required to use a Credential Format in the context of this specification are defined in the Credential Format Profile. Definitions of Credential Formats is out of scope for this specification. Examples for Credential Formats are IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mDL [@ISO.18013-5], and W3C VCDM [@VC_DATA].```

Credential Format Profile:
:  Set of parameters specific to individual Credential Formats. This specification provides Credential Format Profiles for IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mDL [@ISO.18013-5], and W3C VCDM [@VC_DATA], which can be found in section (#format-profiles). Additionally, other specifications or deployments can define their own Credential Format Profiles by utilizing the extension points defined in this specification.

Credential Format Identifier:
:  An identifier to denote a specific Credential Format in the context of this specification. This identifier implies the use of parameters specific to the respective Credential Format Profile.

Credential Configuration:
: Credential Issuer's description of a particular kind of Credential that the Credential Issuer is offering to issue, along with metadata pertaining to the issuance process and the issued Credentials. Each Credential Configuration references a Credential Format and specifies the corresponding parameters given in the Credential Format Profile. Furthermore it includes information about how issuance of a described Credential be requested, information on cryptographic methods and algorithms supported for issuance, and display information to be used by the Wallet. A Credential Configuration is identified by a Credential Configuration Identifier string that is unique to an Issuer. Credential Issuer metadata consists of one or more Credential Configurations.

Presentation:
: Data that is presented to a specific Verifier, derived from one or more Verifiable Credentials that can be from the same or different Credential Issuers. It can be of any Credential Format.

Credential Issuer (or Issuer):
:  An entity that issues Verifiable Credentials. In the context of this specification, the Credential Issuer acts as an OAuth 2.0 Resource Server (see [@!RFC6749]).

Holder:
:  An entity that receives Verifiable Credentials and has control over them to present them to the Verifiers as Presentations.

Verifier:
:  An entity that requests, receives, and validates Presentations.

Issuer-Holder-Verifier Model:
:  Model that facilitates the exchange of claims, where claims are issued as Verifiable Credentials independently of the process of presenting them to Verifiers in the form of Presentations. An issued Verifiable Credential may be used multiple times, although this is not a requirement.

Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential.

Cryptographic Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proving control over the same private key during the issuance and presentation. The mechanism might depend on the Credential Format. For example, in `jwt_vc_json` Credential Format, a VC with Cryptographic Holder Binding contains a public key or a reference to a public key that corresponds to the private key controlled by the Holder.

Claims-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proofing certain claims, e.g., name and date of birth, for example, by presenting another Verifiable Credential. Claims-based Holder Binding allows long-term, cross-device use of a Credential as it does not depend on cryptographic key material stored on a certain device. One example of such a Verifiable Credential could be a Diploma.

Biometrics-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by demonstrating a certain biometric trait, such as fingerprint or face. One example of a Verifiable Credential with Biometrics-based Holder Binding is a mobile driving license [@ISO.18013-5], which contains a portrait of the holder.

Wallet:
:  An entity used by the Holder to request, receive, store, present, and manage Verifiable Credentials and cryptographic key material. There is no single deployment model of a Wallet: Credentials and keys can be stored and managed either locally, through a remote self-hosted service, or via a remote third-party service. In the context of this specification, the Wallet acts as an OAuth 2.0 Client (see [@!RFC6749]) and obtains an Access Token to access an OAuth 2.0 Resource Server (Credential Endpoint).

Deferred Credential Issuance:
:  Issuance of Credentials not directly in the response to a Credential issuance request but following a period of time that can be used to perform certain offline business processes.

# Overview

## Credential Issuer

This specification defines an API for Credential issuance provided by a Credential Issuer. The API is comprised of the following endpoints:

* A mandatory Credential Endpoint from which Credentials can be issued (see (#credential-endpoint)). From this endpoint, one Credential, or multiple Credentials with the same Credential Dataset can be issued in one request.
* An optional Nonce Endpoint from which a fresh `c_nonce` value can be obtained to be used in proof of possession of key material in a subsequent request to the Credential Endpoint (see (#nonce-endpoint)).
* An optional Deferred Credential Endpoint to allow for the deferred delivery of Credentials (see (#deferred-credential-issuance)).
* An optional mechanism for the Credential Issuer to make a Credential Offer to the Wallet to encourage the Wallet to start the issuance flow (see (#credential-offer-endpoint)).
* An optional mechanism for the Credential Issuer to receive from the Wallet notification(s) of the status of the Credential(s) that have been issued.
* A mechanism for the Credential Issuer to publish metadata about the Credentials it is capable of issuing (see (#credential-issuer-metadata)).

The Credential Endpoint may bind an issued Credential to specific cryptographic key material. Credential requests thus should include proof of possession for the key material. Multiple key proof types are supported.

## OAuth 2.0

According to the OAuth 2.0 framework, each Credential Issuer acts as a Resource Server that is protected by an Access Token issued by an Authorization Server, as defined in OAuth 2.0 [@!RFC6749]. The same Authorization Server can protect one or more Credential Issuers. Wallets identify the Authorization Server for a Credential Issuer by referring to the Credential Issuer's metadata (see (#credential-issuer-metadata)).

All OAuth 2.0 Grant Types and extensions mechanisms can be used in conjunction with the Credential issuance API. Aspects not defined in this specification are expected to follow [@!RFC6749]. 

Existing OAuth 2.0 mechanisms are extended as following:

* A new Grant Type "Pre-Authorized Code" is defined to facilitate flows where the preparation of the Credential issuance is conducted before the actual OAuth flow starts (see (#pre-authz-code-flow)).
* A new authorization details [@!RFC9396] type `openid_credential` is defined to convey the details about the Credentials (including Credential Dataset, Credential Formats, and Credential types) the Wallet wants to obtain (see (#authorization-details)).
* Client metadata is used to convey the Wallet's metadata. The new Client metadata parameter `credential_offer_endpoint` is added to allow a Wallet (acting as OAuth 2.0 client) to publish its Credential Offer Endpoint (see (#client-metadata)).
* Authorization Endpoint: The additional parameter `issuer_state` is added to convey state in the context of processing an issuer-initiated Credential Offer (see (#credential-authz-request)). Additional parameters `wallet_issuer` and `user_hint` are added to enable the Credential Issuer to request Verifiable Presentations from the calling Wallet during Authorization Request processing.

## Core Concepts

In the context of this specification, Credential Datasets define the data (claims) about a subject that is to be included in a Credential.
The Credential Format defines how the data (or Dataset) within final Verifiable Credential is organized and secured.
This can include the specific parameters needed to describe the Credential, which are established in the Credential Format Profile.
While in principle independent of each other, the Credential Dataset and the Credential Format can have a relationship in the sense that an Issuer may only offer certain Credential Formats for certain Credential Datasets.

An End-User typically authorizes the issuance of Credentials with a specific Credential Dataset, but does not usually care about the Credential Format. The same Credential Dataset may even be issued in different Credential Formats or with multiple Credential instances.

### Credential Formats and Credential Format Profiles

This specification is Credential Format agnostic and allows implementers to leverage specific capabilities of Credential Formats of their choice.
To this end, extension points to add Credential Format specific parameters in the Credential Issuer metadata, Credential Offer, Authorization Request, and Credential Request are defined.

Credential Format Profiles for IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mDL [@ISO.18013-5], and W3C VCDM [@VC_DATA] are specified in (#format-profiles).
Other specifications or deployments can define their own Credential Format Profiles using the above-mentioned extension points.

### Multiple Credential Issuance

This specification enables the issuance of Verifiable Credentials through the Credential Endpoint. 
A single request message to this endpoint may request the issuance of one or more Verifiable Credentials.

Credentials can vary in their format, including Credential Format Profile-specific parameters, in their contents known as the Credential Dataset, and in the cryptographic data such as Issuer signatures, hashes, and keys used for Cryptographic Holder Binding.
Credentials can therefore vary in the following dimensions:

- Credential Dataset
- Credential Format
- Cryptographic Data

In the context of a single request, all issued Credentials MUST share the same Credential Format and Credential Dataset. However, they MAY contain different Cryptographic Data, such as being bound to different cryptographic keys (e.g., to ensure unlinkability between the Credentials).
To issue multiple Verifiable Credentials with differing Credential Formats or Credential Datasets, multiple requests MUST be sent to the Credential Endpoint.

In the course of the authorization process, the Credential Issuer MAY also request Credential presentation as a means to authenticate or identify the End-User during the issuance flow, as described in (#use-case-5).

### Issuance Flow Variations

The issuance can have multiple characteristics that can be combined depending on the use cases:

* Authorization Code Flow or Pre-Authorized Code Flow: The Credential Issuer can obtain End-User information to turn into a Verifiable Credential using End-User authentication and consent at the Credential Issuer's Authorization Endpoint (Authorization Code Flow) or using out-of-band mechanisms outside of the issuance flow (Pre-Authorized Code Flow).
* Wallet initiated or Issuer initiated: The request from the Wallet can be sent to the Credential Issuer without any gesture from the Credential Issuer (Wallet Initiated) or following the communication from the Credential Issuer (Issuer Initiated).
* Same-device or Cross-device Credential Offer: The End-User may receive the Credential Offer from the Credential Issuer either on the same device as the device the Wallet resides on, or through any other means, such as another device or postal mail, so that the Credential Offer can be communicated to the Wallet.
* Immediate or Deferred: The Credential Issuer can issue the Credential directly in response to the Credential Request (immediate) or requires time and needs the Wallet to come back to retrieve Credential (deferred).

The following subsections illustrate some of the authorization flows supported by this specification.

## Authorization Code Flow {#authorization-code-flow}

The Authorization Code Flow uses the grant type `authorization_code` as defined in [@!RFC6749] to issue Access Tokens.

Figure 1 shows the Authorization Code Flows with the two variations that can be implemented for the issuance of Credentials, as outlined in this specification:

* **Wallet-initiated variation**, described in (#use-case-4) or (#use-case-6);
* **Issuer-initiated variation**, described in (#use-case-1).

Please note that the diagram does not illustrate all the optional features defined in this specification.

!---
~~~ ascii-art
+----------+   +-----------+            +----------------------+     +-------------------+
| End-User |   |   Wallet  |            | Authorization Server |     | Credential Issuer |
+----------+   +-----------+            +----------------------+     +-------------------+
    |                |                              |                        |
    | (1a) End-User  |                              |                        |
    |  selects       |  (1b) Credential Offer       |                        |
    |  Credential--->|  (credential type)           |                        |
    |                |<------------------------------------------------------|
    |                |                              |                        |
    |                |  (2) Obtains Issuer's        |                        |
    |                |      Credential Issuer       |                        |
    |                |      metadata                |                        |
    |                |------------------------------------------------------>|
    |                |                              |                        |
    |                |                              |  (3) Authorization     |
    |                |                              |      Request           |
    |                |                              |      (type(s) of       |
    |                |                              |      Credentials to    |
    |                |                              |      be issued)        |
    |                |----------------------------->|                        |
    |                |                              |                        |
    |  End-User Authentication / Consent            |                        |
    |                |                              |  (4) Authorization     |
    |                |                              |      Response (code)   |
    |                |<-----------------------------|                        |
    |                |                              |                        |
    |                |                              |  (5) Token Request     |
    |                |                              |      (code)            |
    |                |----------------------------->|      Token Response    |
    |                |                              |      (Access Token)    |
    |                |<-----------------------------|                        |
    |                |                              |                        |
    |                |  (6) Credential Request      |                        |
    |                |      (Access Token, proof(s))|                        |
    |                |------------------------------------------------------>|
    |                |                              |                        |
    |                |      Credential Response     |                        |
    |                |      with Credential(s) OR   |                        |
    |                |      Transaction ID          |                        |
    |                |<-----------------------------------------------------|
~~~
!---
Figure: Issuance using Authorization Code Flow 

(1a) The Wallet-initiated flow begins as the End-User requests a Credential via the Wallet from the Credential Issuer. The End-User either selects a Credential from a pre-configured list of Credentials ready to be issued, or alternatively, the Wallet gives guidance to the End-User to select a Credential from a Credential Issuer based on the information it received in the presentation request from a Verifier.

(1b) The Issuer-initiated flow begins as the Credential Issuer generates a Credential Offer for certain Credential(s) that it communicates to the Wallet, for example, as a QR code or as a URI. The Credential Offer contains the Credential Issuer's URL and the information about the Credential(s) being offered. This step is defined in (#credential-offer).

(2) The Wallet uses the Credential Issuer's URL to fetch the Credential Issuer metadata, as described in (#credential-issuer-metadata). The Wallet needs the metadata to learn the Credential types and formats that the Credential Issuer supports and to determine the Authorization Endpoint (OAuth 2.0 Authorization Server) as well as Credential Endpoint required to start the request. This specification supports configurations where the Credential Endpoint and the Authorization Endpoint are managed by either separate entities or a single entity.

(3) The Wallet sends an Authorization Request to the Authorization Endpoint. The Authorization Endpoint processes the Authorization Request, which typically includes authenticating the End-User and gathering End-User consent. Note: The Authorization Request may be sent as a Pushed Authorization Request.

Note: Steps (3) and (4) happen in the front channel, by redirecting the End-User via the User Agent. Those steps are defined in (#authorization-endpoint). The Authorization Server and the User Agent may exchange any further messages between the steps if required by the Authorization Server to authenticate the End-User.

(4) The Authorization Endpoint returns the Authorization Response with the Authorization Code upon successfully processing the Authorization Request.

(5) The Wallet sends a Token Request to the Token Endpoint with the Authorization Code obtained in Step (4). The Token Endpoint returns an Access Token in the Token Response upon successfully validating the Authorization Code. This step happens in the back-channel communication (direct communication between two systems using HTTP requests and responses without using redirects through an intermediary such as a browser). This step is defined in (#token-endpoint).

(6) The Wallet sends a Credential Request to the Credential Issuer's Credential Endpoint with the Access Token and (optionally) the proof of possession of the private key of a key pair to which the Credential Issuer should bind the issued Credential to. Upon successfully validating Access Token and proof, the Credential Issuer returns a Credential in the Credential Response. This step is defined in (#credential-endpoint).

If the Credential Issuer requires more time to issue a Credential, the Credential Issuer may return a Transaction ID and a time interval in the Credential Response. The Wallet may send a Deferred Credential Request with the Transaction ID to obtain a Credential after the specified time interval has passed, as defined in (#deferred-credential-issuance).

Note: This flow is based on OAuth 2.0 and the Authorization Code Grant type, but this specification can be used with other OAuth 2.0 grant types as well.

## Pre-Authorized Code Flow {#pre-authz-code-flow}

Figure 2 is a diagram of a Credential issuance using the Pre-Authorized Code Flow. In this flow, before initiating the flow with the Wallet, the Credential Issuer first conducts the steps required to prepare for Credential issuance, e.g., End-User authentication and authorization. Consequently, the Pre-Authorized Code is sent by the Credential Issuer to the Wallet. This flow does not use the Authorization Endpoint, and the Wallet exchanges the Pre-Authorized Code for the Access Token directly at the Token Endpoint. The Access Token is then used to request Credential issuance at the Credential Endpoint. See (#use-case-2) for the description of such a use case.

How the End-User provides information required for the issuance of a requested Credential to the Credential Issuer and the business processes conducted by the Credential Issuer to prepare a Credential are out of scope of this specification.

This flow uses the OAuth 2.0 Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`, which is defined in (#credential-offer-parameters).

The following diagram is based on the Credential Issuer-initiated flow, as described in the use case in (#use-case-2). Please note that it does not illustrate all the optional features outlined in this specification.

!---
~~~ ascii-art
+--------------+   +-----------+            +----------------------+   +-------------------+
|   End-User   |   |   Wallet  |            | Authorization Server |   | Credential Issuer |
+--------------+   +-----------+            +----------------------+   +-------------------+
        |                |                              |                        |
        |                |  (1) End-User provides       |                        |
        |                |      information required    |                        |
        |                |      for the issuance of     |                        |
        |                |      a certain Credential    |                        |
        |                |------------------------------------------------------>|
        |                |                              |                        |
        |                |  (2) Credential Offer        |                        |
        |                |      (Pre-Authorized Code)   |                        |
        |                |<------------------------------------------------------|
        |                |  (3) Obtains Issuer's        |                        |
        |                |      Credential Issuer       |                        |
        |                |      metadata                |                        |
        |                |------------------------------------------------------>|
        |   interacts    |                              |                        |
        |--------------->|                              |                        |
        |                |                              |                        |
        |                |  (4) Token Request           |                        |
        |                |      (Pre-Authorized Code,   |                        |
        |                |       tx_code)               |                        |
        |                |----------------------------->|                        |
        |                |      Token Response          |                        |
        |                |      (access_token)          |                        |
        |                |<-----------------------------|                        |
        |                |                              |                        |
        |                |  (5) Credential Request      |                        |
        |                |      (access_token, proof(s))|                        |
        |                |------------------------------------------------------>|
        |                |      Credential Response     |                        |
        |                |      (Credential(s))         |                        |
        |                |<------------------------------------------------------|         
~~~
!---
Figure: Issuance using Pre-Authorized Code Flow 

(1) The Credential Issuer successfully obtains consent and End-User data required for the issuance of a requested Credential from the End-User using an Issuer-specific business process.

(2) The flow defined in this specification begins as the Credential Issuer generates a Credential Offer for certain Credential(s) and communicates it to the Wallet, for example, as a QR code or as a URI. The Credential Offer contains the Credential Issuer's URL, the information about the Credential(s) being offered, and the Pre-Authorized Code. This step is defined in (#credential-offer).

(3) The Wallet uses the Credential Issuer's URL to fetch its metadata, as described in (#credential-issuer-metadata). The Wallet needs the metadata to learn the Credential types and formats that the Credential Issuer supports, and to determine the Token Endpoint (at the OAuth 2.0 Authorization Server) as well as the Credential Endpoint required to start the request.

(4) The Wallet sends the Pre-Authorized Code obtained in Step (2) in the Token Request to the Token Endpoint. The Wallet will additionally send a Transaction Code provided by the End-User, if it was required by the Credential Issuer. This step is defined in (#token-endpoint).

(5) This step is the same as Step (6) in the Authorization Code Flow.

It is important to note that anyone who possesses a valid Pre-Authorized Code, without further security measures, would be able to receive a VC from the Credential Issuer. Implementers MUST implement mitigations most suitable to the use case.

One such mechanism defined in this specification is the usage of Transaction Codes. The Credential Issuer indicates the usage of Transaction Codes in the Credential Offer and sends the Transaction Code to the End-User via a second channel different than the issuance flow. After the End-User provides the Transaction Code, the Wallet sends the Transaction Code within the Token Request, and the Authorization Server verifies the Transaction Code.

For more details and concrete mitigations, see (#security-considerations-pre-authz-code).

# Credential Offer Endpoint {#credential-offer-endpoint}

This endpoint is used by a Credential Issuer that is already interacting with an End-User who wishes to initiate a Credential issuance. It is used to pass available information relevant for the Credential issuance to ensure a convenient and secure process.

## Credential Offer {#credential-offer}

The Credential Issuer makes a Credential Offer by allowing the End-User to invoke the Wallet using the Wallet's Credential Offer Endpoint defined in (#client-metadata). For example, by clicking a link and/or rendering a QR code containing the Credential Offer that the End-User can scan in a wallet or an arbitrary camera application.

Credential Issuers MAY also communicate Credential Offers directly to a Wallet's backend but any mechanism for doing so is currently outside the scope of this specification.

The Credential Offer object, which is a JSON-encoded object with the Credential Offer parameters, can be sent by value or by reference.

The Credential Offer contains a single URI query parameter, either `credential_offer` or `credential_offer_uri`:

* `credential_offer`: Object with the Credential Offer parameters. This MUST NOT be present when the `credential_offer_uri` parameter is present.
* `credential_offer_uri`: String that is a URL using the `https` scheme referencing a resource containing a JSON object with the Credential Offer parameters. This MUST NOT be present when the `credential_offer` parameter is present.

For security considerations, see (#credential-offer-security).

### Credential Offer Parameters {#credential-offer-parameters}

This specification defines the following parameters for the JSON-encoded Credential Offer object:

* `credential_issuer`: REQUIRED. The URL of the Credential Issuer, as defined in (#credential-issuer-identifier), from which the Wallet is requested to obtain one or more Credentials. The Wallet uses it to obtain the Credential Issuer's Metadata following the steps defined in (#credential-issuer-wellknown).
* `credential_configuration_ids`: REQUIRED. Array of unique strings that each identify one of the keys in the name/value pairs stored in the `credential_configurations_supported` Credential Issuer metadata. The Wallet uses these string values to obtain the respective object that contains information about the Credential being offered as defined in (#credential-issuer-parameters). For example, these string values can be used to obtain `scope` values to be used in the Authorization Request.
* `grants`: OPTIONAL. Object indicating to the Wallet the Grant Types the Credential Issuer's Authorization Server is prepared to process for this Credential Offer. Every grant is represented by a name/value pair. The name is the Grant Type identifier; the value is an object that contains parameters either determining the way the Wallet MUST use the particular grant and/or parameters the Wallet MUST send with the respective request(s). If `grants` is not present or is empty, the Wallet MUST determine the Grant Types the Credential Issuer's Authorization Server supports using the respective metadata. When multiple grants are present, it is at the Wallet's discretion which one to use.

Additional Credential Offer parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

The following values are defined by this specification: 

* Grant Type `authorization_code`:
  * `issuer_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the Wallet that is used to bind the subsequent Authorization Request with the Credential Issuer to a context set up during previous steps. If the Wallet decides to use the Authorization Code Flow and received a value for this parameter, it MUST include it in the subsequent Authorization Request to the Credential Issuer as the `issuer_state` parameter value.
  * `authorization_server`: OPTIONAL string that the Wallet can use to identify the Authorization Server to use with this grant type when `authorization_servers` parameter in the Credential Issuer metadata has multiple entries. It MUST NOT be used otherwise. The value of this parameter MUST match with one of the values in the `authorization_servers` array obtained from the Credential Issuer metadata.
* Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`:
  * `pre-authorized_code`: REQUIRED. The code representing the Credential Issuer's authorization for the Wallet to obtain Credentials of a certain type. This code MUST be short lived and single use. If the Wallet decides to use the Pre-Authorized Code Flow, this parameter value MUST be included in the subsequent Token Request with the Pre-Authorized Code Flow.
  * `tx_code`: OPTIONAL. Object describing the requirements for a Transaction Code, which the Authorization Server expects the End-User to present along with the Token Request in a Pre-Authorized Code Flow. If the Authorization Server does not expect a Transaction Code, this object is absent; this is the default. The Transaction Code is intended to bind the Pre-Authorized Code to a certain transaction to prevent replay of this code by an attacker that, for example, scanned the QR code while standing behind the legitimate End-User. It is RECOMMENDED to send the Transaction Code via a separate channel. If the Wallet decides to use the Pre-Authorized Code Flow, the Transaction Code value MUST be sent in the `tx_code` parameter with the respective Token Request as defined in (#token-request). If no `length` or `description` is given, this object may be empty, indicating that a Transaction Code is required.
    * `input_mode` : OPTIONAL. String specifying the input character set. Possible values are `numeric` (only digits) and `text` (any characters). The default is `numeric`.
    * `length`: OPTIONAL. Integer specifying the length of the Transaction Code. This helps the Wallet to render the input screen and improve the user experience.
    * `description`: OPTIONAL. String containing guidance for the Holder of the Wallet on how to obtain the Transaction Code, e.g., describing over which communication channel it is delivered. The Wallet is RECOMMENDED to display this description next to the Transaction Code input screen to improve the user experience. The length of the string MUST NOT exceed 300 characters. The `description` does not support internationalization, however the Issuer MAY detect the Holder's language by previous communication or an HTTP Accept-Language header within an HTTP GET request for a Credential Offer URI.
  * `authorization_server`: OPTIONAL string that the Wallet can use to identify the Authorization Server to use with this grant type when `authorization_servers` parameter in the Credential Issuer metadata has multiple entries. It MUST NOT be used otherwise. The value of this parameter MUST match with one of the values in the `authorization_servers` array obtained from the Credential Issuer metadata.
  
The following non-normative example shows a Credential Offer object where the Credential Issuer can offer the issuance of two different Credentials (which may even be of different formats):

<{{examples/credential_offer_multiple_credentials.json}}

### Sending Credential Offer by Value Using `credential_offer` Parameter

Below is a non-normative example of a Credential Offer passed by value:

```
GET /credential_offer?credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.example.com%22,%22credential_configuration_ids%22:%5B%22UniversityDegree_JWT%22,%22org.iso.18013.5.1.mDL%22%5D,%22grants%22:%7B%22urn:ietf:params:oauth:grant-type:pre-authorized_code%22:%7B%22pre-authorized_code%22:%22oaKazRN8I0IbtZ0C7JuMn5%22,%22tx_code%22:%7B%7D%7D%7D%7D
```

The following is a non-normative example of a Credential Offer that can be included in a QR code or a link used to invoke a Wallet deployed as a native app:

```
openid-credential-offer://?credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.example.com%22,%22credential_configuration_ids%22:%5B%22org.iso.18013.5.1.mDL%22%5D,%22grants%22:%7B%22urn:ietf:params:oauth:grant-type:pre-authorized_code%22:%7B%22pre-authorized_code%22:%22oaKazRN8I0IbtZ0C7JuMn5%22,%22tx_code%22:%7B%22input_mode%22:%22text%22,%22description%22:%22Please%20enter%20the%20serial%20number%20of%20your%20physical%20drivers%20license%22%7D%7D%7D%7D
```

### Sending Credential Offer by Reference Using `credential_offer_uri` Parameter

Upon receipt of the `credential_offer_uri`, the Wallet MUST send an HTTP GET request to the URI to retrieve the referenced Credential Offer Object, unless it is already cached, and parse it to recreate the Credential Offer parameters.

Note: The Credential Issuer SHOULD use a unique URI for each Credential Offer utilizing distinct parameters, or otherwise prevent the Credential Issuer from caching the `credential_offer_uri`.

Below is a non-normative example of this fetch process:

```
GET /credential_offer HTTP/1.1
Host: server.example.com
```

The response from the Credential Issuer that contains a Credential Offer Object MUST use the media type `application/json`.

This ability to pass the Credential Offer by reference is particularly useful for large Credential Offer objects.

When the Credential Offer is displayed as a QR code, it would usually contain the Credential Offer by reference due to the size limitations of the QR codes. Below is a non-normative example:

```
openid-credential-offer://?
  credential_offer_uri=https%3A%2F%2Fserver%2Eexample%2Ecom%2Fcredential-offer
  %2FGkurKxf5T0Y-mnPFCHqWOMiZi4VS138cQO_V7PZHAdM
```

Below is a non-normative example of a response from the Credential Issuer that contains a Credential Offer Object used to encourage the Wallet to start an Authorization Code Flow:

<{{examples/credential_offer_authz_code.txt}}

Below is a non-normative example of a Credential Offer Object for a Pre-Authorized Code Flow (with a Credential type reference):

<{{examples/credential_offer_by_reference.json}}

When retrieving the Credential Offer from the Credential Offer URL, the `application/json` media type MUST be used. The Credential Offer cannot be signed and MUST NOT use `application/jwt` with `"alg": "none"`.

## Credential Offer Response

The Wallet does not create a response. UX control stays with the Wallet after completion of the process. 

# Authorization Endpoint {#authorization-endpoint}

The Authorization Endpoint is used in the same manner as defined in [@!RFC6749], taking into account the recommendations given in [@!I-D.ietf-oauth-security-topics].

When the grant type `authorization_code` is used, it is RECOMMENDED to use PKCE [@!RFC7636] and Pushed Authorization Requests [@RFC9126]. PKCE prevents authorization code interception attacks. Pushed Authorization Requests ensure the integrity and authenticity of the authorization request.

## Authorization Request {#credential-authz-request}

An Authorization Request is an OAuth 2.0 Authorization Request as defined in Section 4.1.1 of [@!RFC6749], which requests that access be granted to the Credential Endpoint, as defined in (#credential-endpoint).

There are two possible methods for requesting the issuance of a specific Credential type in an Authorization Request. The first method involves using the `authorization_details` request parameter, as defined in [@!RFC9396], containing one or more authorization details of type `openid_credential`, as specified in (#authorization-details). The second method utilizes scopes, as outlined in (#credential-request-using-type-specific-scope).

### Using Authorization Details Parameter {#authorization-details}

Credential Issuers MAY support requesting authorization to issue a Credential using the `authorization_details` parameter.

The request parameter `authorization_details` defined in Section 2 of [@!RFC9396] MUST be used to convey the details about the Credentials the Wallet wants to obtain. This specification introduces a new authorization details type `openid_credential` and defines the following parameters to be used with this authorization details type:

* `type`: REQUIRED. String that determines the authorization details type. It MUST be set to `openid_credential` for the purpose of this specification.
* `credential_configuration_id`: REQUIRED when `format` parameter is not present. String specifying a unique identifier of the Credential being described in the `credential_configurations_supported` map in the Credential Issuer Metadata as defined in (#credential-issuer-parameters). The referenced object in the `credential_configurations_supported` map conveys the details, such as the format, for issuance of the requested Credential. This specification defines Credential Format specific Issuer Metadata in (#format-profiles). It MUST NOT be present if `format` parameter is present.
* `format`: REQUIRED when `credential_configuration_id` parameter is not present. String identifying the format of the Credential the Wallet needs. This Credential Format Identifier determines further claims in the authorization details object needed to identify the Credential type in the requested format. This specification defines Credential Format Profiles in (#format-profiles). It MUST NOT be present if `credential_configuration_id` parameter is present.

Additional `authorization_details` data fields MAY be defined and used
when the `type` value is `openid_credential`.
Note that this effectively defines an authorization details type that is never considered invalid due to unknown fields.

The following is a non-normative example of an `authorization_details` object with a `credential_configuration_id`:

<{{examples/authorization_details.json}}

The following is a non-normative example of an `authorization_details` object requesting a Credential in the IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format:

<{{examples/authorization_details_sd_jwt_vc.json}}

If the Credential Issuer metadata contains an `authorization_servers` parameter, the authorization detail's `locations` common data field MUST be set to the Credential Issuer Identifier value. A non-normative example for a deployment where an Authorization Server protects multiple Credential Issuers would look like this:

<{{examples/authorization_details_with_as.json}}

Below is a non-normative example of an Authorization Request using the `authorization_details` parameter that would be sent by the User Agent to the Authorization Server in response to an HTTP 302 redirect response by the Wallet (with line breaks within values for display purposes only):

```
GET /authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &authorization_details=%5B%7B%22type%22%3A%20%22openid_credential%22%2C%20%22
    credential_configuration_id%22%3A%20%22UniversityDegreeCredential%22%7D%5D
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
  
Host: server.example.com
```

This non-normative example requests authorization to issue two different Credentials:

<{{examples/authorization_details_multiple_credentials.json}}

Note: Applications MAY combine authorization details of type `openid_credential` with any other authorization details types in an Authorization Request.

### Using `scope` Parameter to Request Issuance of a Credential {#credential-request-using-type-specific-scope}

Credential Issuers MAY support requesting authorization to issue a Credential using the OAuth 2.0 `scope` parameter.

When the Wallet does not know which scope value to use to request issuance of a certain Credential, it can discover it using the `scope` Credential Issuer metadata parameter defined in (#credential-issuer-parameters). When the flow starts with a Credential Offer, the Wallet can use the `credential_configuration_ids` parameter values to identify object(s) in the `credential_configurations_supported` map in the Credential Issuer metadata parameter and use the `scope` parameter value from that object.

The Wallet can discover the scope values using other options such as normative text in a profile of this specification that defines scope values along with a description of their semantics.

The concrete `scope` values are out of scope of this specification.

The Wallet MAY combine scopes discovered from the Credential Issuer metadata with the scopes discovered from the Authorization Server metadata.

It is RECOMMENDED to use collision-resistant scope values.

Credential Issuers MUST interpret each scope value as a request to access the Credential Endpoint as defined in (#credential-endpoint) for the issuance of a Credential type identified by that scope value. Multiple scope values MAY be present in a single request whereby each 
occurrence MUST be interpreted individually.

Credential Issuers MUST ignore unknown scope values in a request.

If the Credential Issuer metadata contains an `authorization_servers` property, it is RECOMMENDED to use a `resource` parameter [@!RFC8707] whose value is the Credential Issuer's identifier value to allow the Authorization Server to differentiate Credential Issuers.  

Below is a non-normative example of an Authorization Request provided by the Wallet to the Authorization Server using the scope `UniversityDegreeCredential` and in response to an HTTP 302 redirect (with line breaks within values for display purposes only):

```
GET /authorize?
  response_type=code
  &scope=UniversityDegreeCredential
  &resource=https%3A%2F%2Fcredential-issuer.example.com
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
Host: server.example.com
```

If a scope value related to Credential issuance and the `authorization_details` request parameter containing objects of type `openid_credential` are both present in a single request, the Credential Issuer MUST interpret these individually. However, if both request the same Credential type, then the Credential Issuer MUST follow the request as given by the authorization details object.

### Additional Request Parameters {#additional-request-parameters}

This specification defines the following request parameters that can be supplied in an Authorization Request:

* `wallet_issuer`: OPTIONAL. String containing the Wallet's identifier. The Credential Issuer can use the discovery process defined in [@!SIOPv2] to determine the Wallet's capabilities and endpoints, using the `wallet_issuer` value as the Issuer Identifier referred to in [@!SIOPv2]. This is RECOMMENDED in Dynamic Credential Requests.
* `user_hint`: OPTIONAL. String containing an opaque End-User hint that the Wallet MAY use in subsequent callbacks to optimize the End-User's experience. This is RECOMMENDED in Dynamic Credential Requests.
* `issuer_state`: OPTIONAL. String value identifying a certain processing context at the Credential Issuer. A value for this parameter is typically passed in a Credential Offer from the Credential Issuer to the Wallet (see (#credential-offer)). This request parameter is used to pass the `issuer_state` value back to the Credential Issuer.

Note: When processing the Authorization Request, the Credential Issuer MUST take into account that the `issuer_state` is not guaranteed to originate from this Credential Issuer in all circumstances. It could have been injected by an attacker.

Additional Authorization Request parameters MAY be defined and used,
as described in [@!RFC6749].
The Authorization Server MUST ignore any unrecognized parameters.

### Pushed Authorization Request

Use of Pushed Authorization Requests is RECOMMENDED to ensure confidentiality, integrity, and authenticity of the request data and to avoid issues caused by large requests sizes.

Below is a non-normative example of a Pushed Authorization Request:

```
POST /op/par HTTP/1.1
Host: as.example.com
Content-Type: application/x-www-form-urlencoded

response_type=code
&client_id=CLIENT1234
&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
&code_challenge_method=S256
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
&authorization_details=...
```

### Dynamic Credential Request

This step is OPTIONAL. After receiving an Authorization Request from the Client, the Credential Issuer MAY use this step to obtain additional Credentials from the End-User required to proceed with the authorization of the Credential issuance. The Credential Issuer MAY obtain a Credential and utilize it to identify the End-User before issuing an additional Credential. For such a use case, see (#use-case-5).

It is RECOMMENDED that the Credential Issuer use [@OpenID4VP] to dynamically request presentation of additional Credentials. From a protocol perspective, the Credential Issuer then acts as a verifier and sends a presentation request to the Wallet. The Client SHOULD obtain these Credentials prior to starting a transaction with this Credential Issuer.

To enable dynamic callbacks of the Credential Issuer to the End-User's Wallet, the Wallet MAY provide the additional parameters `wallet_issuer` and `user_hint` defined in (#additional-request-parameters).

For non-normative examples of the request and response, see Sections 5 and 6 in [@OpenID4VP].

Note to the editors: We need to sort out the Credential Issuer's `client_id` with the Wallet and potentially add an example with `wallet_issuer` and `user_hint`.

## Successful Authorization Response {#authorization-response}

Authorization Responses MUST be made as defined in [@!RFC6749].

Below is a non-normative example of a successful Authorization Response:

```
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?
  code=SplxlOBeZQQYbYS6WxSbIA
```

## Authorization Error Response

The Authorization Error Response MUST be made as defined in [@!RFC6749].

Below is a non-normative example of an unsuccessful Authorization Response.

```json=
HTTP/1.1 302 Found
Location: https://client.example.net/cb?
  error=invalid_request
  &error_description=Unsupported%20response_type%20value
```

# Token Endpoint {#token-endpoint}

The Token Endpoint issues an Access Token and, optionally, a Refresh Token in exchange for the Authorization Code that Client obtained in a successful Authorization Response. It is used in the same manner as defined in [@!RFC6749] and follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

## Token Request {#token-request}

The Token Request is made as defined in Section 4.1.3 of [@!RFC6749].

The following are the extension parameters to the Token Request used in the Pre-Authorized Code Flow defined in (#pre-authz-code-flow):

* `pre-authorized_code`: The code representing the authorization to obtain Credentials of a certain type. This parameter MUST be present if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.
* `tx_code`: OPTIONAL. String value containing a Transaction Code value itself. This value MUST be present if a `tx_code` object was present in the Credential Offer (including if the object was empty). This parameter MUST only be used if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.

Requirements around how the Wallet identifies and, if applicable, authenticates itself with the Authorization Server in the Token Request depends on the Client type defined in Section 2.1 of [@!RFC6749] and the Client authentication method indicated in the `token_endpoint_auth_method` Client metadata. The requirements specified in Sections 4.1.3 and 3.2.1 of [@!RFC6749] MUST be followed.

For the Pre-Authorized Code Grant Type, authentication of the Client is OPTIONAL, as described in Section 3.2.1 of OAuth 2.0 [@!RFC6749], and, consequently, the `client_id` parameter is only needed when a form of Client Authentication that relies on this parameter is used.

If the Token Request contains an `authorization_details` parameter (as defined by [@!RFC9396]) of type `openid_credential` and the Credential Issuer's metadata contains an `authorization_servers` parameter, the `authorization_details` object MUST contain the Credential Issuer's identifier in the `locations` element. 

If the Token Request contains a scope value related to Credential issuance and the Credential Issuer's metadata contains an `authorization_servers` parameter, it is RECOMMENDED to use a `resource` parameter [@!RFC8707] whose value is the Credential Issuer's identifier value to allow the Authorization Server to differentiate Credential Issuers.

When the Pre-Authorized Grant Type is used, it is RECOMMENDED that the Credential Issuer issues an Access Token valid only for the Credentials indicated in the Credential Offer (see (#credential-offer)). The Wallet SHOULD obtain a separate Access Token if it wants to request issuance of any Credentials that were not included in the Credential Offer, but were discoverable from the Credential Issuer's `credential_configurations_supported` metadata parameter.

Additional Token Request parameters MAY be defined and used,
as described in [@!RFC6749].
The Authorization Server MUST ignore any unrecognized parameters.

Below is a non-normative example of a Token Request in an Authorization Code Flow:

```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
&redirect_uri=https%3A%2F%2FWallet.example.org%2Fcb
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSU...
```

### Request Issuance of a Certain Credential using authorization_details Parameter

Credential Issuers MAY support requesting authorization to issue a Credential using the `authorization_details` parameter. This is particularly useful, if the Credential Issuer offered multiple Credential Configurations in the Credential Offer of a Pre-Authorized Code Flow. 

The Wallet can use `authorization_details` in the Token Request to request a specific Credential Configuration in both the Authorization Code Flow and the Pre-Authorized Code Flow. The value of the `authorization_details` parameter is defined in (#(authorization-details)).

Below is a non-normative example of a Token Request in a Pre-Authorized Code Flow (without Client Authentication):

```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
&pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
&tx_code=493536
&authorization_details=%5B%7B%22type%22%3A%20%22openid_credential%22%2C%20%22
    credential_configuration_id%22%3A%20%22UniversityDegreeCredential%22%7D%5D
```

## Successful Token Response {#token-response}

Token Responses are made as defined in [@!RFC6749].

The Authorization Server might decide to authorize issuance of multiple instances for each Credential requested in the Authorization Request. Each Credential instance is described using the same entry in the `credential_configurations_supported` Credential Issuer metadata, but contains different claim values or different subset of claims within the claims set identified by the `credential_configuration_id`.

In addition to the response parameters defined in [@!RFC6749], the Authorization Server MAY return the following parameters:

* `authorization_details`: REQUIRED when the `authorization_details` parameter is used to request issuance of a certain Credential Configuration as defined in (#authorization-details). It MUST NOT be used otherwise. It is an array of objects, as defined in Section 7 of [@!RFC9396]. In addition to the parameters defined in (#authorization-details), this specification defines the following parameter to be used with the authorization details type `openid_credential` in the Token Response:
  * `credential_identifiers`: REQUIRED. Array of strings, each uniquely identifying a Credential Dataset that can be issued using the Access Token returned in this response. Each of these Credential Datasets corresponds to the same Credential Configuration in the `credential_configurations_supported` parameter of the Credential Issuer metadata. The Wallet MUST use these identifiers together with an Access Token in subsequent Credential Requests.

Additional Token Response parameters MAY be defined and used,
as described in [@!RFC6749].
The Wallet MUST ignore any unrecognized parameters in the Token Response.
An included `authorization_details` parameter MAY also have additional data fields defined and used
when the `type` value is `openid_credential`.
The Wallet MUST ignore any unrecognized data fields in the `authorization_details` present in the Token Response.

Below is a non-normative example of a Token Response when the `authorization_details` parameter was used to request issuance of a certain Credential type:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
  "token_type": "bearer",
  "expires_in": 86400,
  "authorization_details": [
    {
      "type": "openid_credential",
      "credential_configuration_id": "UniversityDegreeCredential",
      "credential_identifiers": [ "CivilEngineeringDegree-2023", "ElectricalEngineeringDegree-2023" ]
    }
  ]
}
```

## Token Error Response {#token-error-response}

If the Token Request is invalid or unauthorized, the Authorization Server constructs the error response as defined as in Section 5.2 of OAuth 2.0 [@!RFC6749].

The following additional clarifications are provided for some of the error codes already defined in [@!RFC6749]:

`invalid_request`:

- The Authorization Server does not expect a Transaction Code in the Pre-Authorized Code Flow but the Client provides a Transaction Code.
- The Authorization Server expects a Transaction Code in the Pre-Authorized Code Flow but the Client does not provide a Transaction Code.

`invalid_grant`:

- The Authorization Server expects a Transaction Code in the Pre-Authorized Code Flow but the Client provides the wrong Transaction Code.
- The End-User provides the wrong Pre-Authorized Code or the Pre-Authorized Code has expired.

`invalid_client`:

- The Client tried to send a Token Request with a Pre-Authorized Code without a Client ID but the Authorization Server does not support anonymous access.

Below is a non-normative example Token Error Response:

```json=
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "invalid_request"
}
```

# Nonce Endpoint {#nonce-endpoint}

This endpoint allows a Client to acquire a fresh `c_nonce` value. A Credential Issuer that requires `c_nonce` values to be incorporated into proofs in the Credential Request (see (#credential-request)) MUST offer a Nonce Endpoint.

The `nonce_endpoint` Credential Issuer Metadata parameter, as defined in (#credential-issuer-parameters), contains the URL of the Credential Issuer's Nonce Endpoint.


## Nonce Request {#nonce-request}

A request for a nonce is made by sending an HTTP POST request to the URL provided in the `nonce_endpoint` Credential Issuer Metadata parameter.

Below is a non-normative example of a Nonce Request:

```
POST /nonce HTTP/1.1
Host: credential-issuer.example.com
Content-Length: 0
```

## Nonce Response {#nonce-response}

The Credential Issuer provides a nonce value in the HTTP response with a 2xx status code and the following parameters included as top-level members in the message body of the HTTP response using the application/json media type:

* `c_nonce`: REQUIRED. String containing a nonce to be used when creating a proof of possession of the key proof (see (#credential-request)).
* `c_nonce_expires_in`: OPTIONAL. Number denoting the lifetime in seconds of the `c_nonce`. This value serves only as a hint to the Client, indicating how long the Credential Issuer is likely to accept the `c_nonce` as valid.

Due to the temporal and contextually sensitive nature of the `c_nonce` value, the Credential Issuer MUST make the response uncacheable by adding a `Cache-Control` header field including the value `no-store`.

Below is a non-normative example of a Nonce Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "c_nonce": "wKI4LT17ac15ES9bw8ac4",
  "c_nonce_expires_in": 120
}
```


# Credential Endpoint {#credential-endpoint}

The Credential Endpoint issues one or more Credentials of the same Credential Configuration and Credential Dataset (as approved by the End-User) upon presentation of a valid Access Token representing this approval. Support for this endpoint is REQUIRED.

Communication with the Credential Endpoint MUST utilize TLS.

The Client sends a Credential Request to obtain:

* one Credential; or
* multiple Credential instances of the same Credential Configuration and Credential Dataset, each with distinct cryptographic material.

If the Issuer supports the issuance of multiple Credentials, the Client can send several consecutive Credential Requests to obtain multiple Credentials in a chosen sequence.

## Binding the Issued Credential to the Identifier of the End-User Possessing that Credential {#credential-binding}

The issued Credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the Credential. Cryptographic binding allows the Verifier to verify during the presentation of a Credential that the End-User presenting a Credential is the same End-User to whom that Credential was issued. For non-cryptographic types of binding and Credentials issued without any binding, see the Implementation Considerations in (#claims-based-binding) and (#no-binding).

Note: Claims in the Credential are about the subject of the Credential, which is often the End-User who possesses it.

For cryptographic binding, the Client has the following options defined in (#credential-request) to provide cryptographic binding material for a requested Credential:

1. Provide proof of control alongside key material.
1. Provide only proof of control without the key material.

## Credential Request {#credential-request}

A Client makes a Credential Request to the Credential Endpoint by sending the following parameters in the entity-body of an HTTP POST request using the `application/json` media type.

* `credential_identifier`: REQUIRED when an Authorization Details of type `openid_credential` was returned from the Token Response. It MUST NOT be used otherwise. A string that identifies a Credential Dataset that is requested for issuance. When this parameter is used, the `format` parameter and any other Credential format specific parameters such as those defined in (#format-profiles) MUST NOT be present.
* `format`: REQUIRED if an Authorization Details of type `openid_credential` was not returned from the Token Response (e.g. when the credential was requested using a `scope` value in the authorization request or a pre-authorisation code was used that did not return an Authorization Details). It MUST NOT be used otherwise. A string that determines the format of the Credential to be issued, which may determine the type and any other information related to the Credential to be issued. Credential Format Profiles consist of the Credential format specific parameters that are defined in (#format-profiles). When this parameter is used, the `credential_identifier` Credential Request parameter MUST NOT be present.
* `proof`: OPTIONAL. Object providing a single proof of possession of the cryptographic key material to which the issued Credential instance will be bound to. `proof` parameter MUST NOT be present if `proofs` parameter is used.  The `proof` object MUST contain the following:
    * `proof_type`: REQUIRED. String specifying the key proof type. The value set for this parameter determines the additional parameters in the key proof object and their corresponding processing rules. The key proof types outlined in this specification are detailed in (#proof-types).
* `proofs`: OPTIONAL. Object providing one or more proof of possessions of the cryptographic key material to which the issued Credential instances will be bound to. The `proofs` parameter MUST NOT be present if `proof` parameter is used. `proofs` object contains exactly one parameter named as the proof type in (#proof-types), the value set for this parameter is an array containing parameters as defined by the corresponding proof type.
* `credential_response_encryption`: OPTIONAL. Object containing information for encrypting the Credential Response. If this request element is not present, the corresponding credential response returned is not encrypted.
    * `jwk`: REQUIRED. Object containing a single public key as a JWK used for encrypting the Credential Response.
    * `alg`: REQUIRED. JWE [@!RFC7516] `alg` algorithm [@!RFC7518] for encrypting Credential Responses.
    * `enc`: REQUIRED. JWE [@!RFC7516] `enc` algorithm [@!RFC7518] for encrypting Credential Responses.

The `proof_type` parameter is an extension point that enables the use of different types of proofs for different cryptographic schemes.

The proof(s) in the `proof` or  `proofs` parameter MUST incorporate the Credential Issuer Identifier (audience), and optionally a `c_nonce` value generated by the Credential Issuer to allow the Credential Issuer to detect replay. The way that data is incorporated depends on the key proof type. In a JWT, for example, the `c_nonce` value is conveyed in the `nonce` claim, whereas the audience is conveyed in the `aud` claim. In a Linked Data proof, for example, the `c_nonce` is included as the `challenge` element in the key proof object and the Credential Issuer (the intended audience) is included as the `domain` element.

The `c_nonce` value can be retrieved from the Nonce Endpoint as defined in (#nonce-endpoint).

Additional Credential Request parameters MAY be defined and used.
The Credential Issuer MUST ignore any unrecognized parameters.

Below is a non-normative example of a Credential Request for a Credential in [@ISO.18013-5] format using Credential Format-specific parameters and a key proof type `jwt`:

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "format":"mso_mdoc",
  "doctype":"org.iso.18013.5.1.mDL",
  "proof": {
    "proof_type": "jwt",
    "jwt": "..."
  }
}
```

Either the `proof` or `proofs` parameter MUST be present if the `proof_types_supported` parameter is present in the `credential_configurations_supported` parameter of the Issuer metadata for the requested Credential.

Below is a non-normative example of a Credential Request for two Credential instances in an IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format using a Credential instance identifier and key proof type `jwt`:

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "credential_identifier": "CivilEngineeringDegree-2023",
  "proofs": {
    "jwt": [
      "eyJ0eXAiOiJvcGVuaWQ0dmNpL...Lb9zioZoipdP-jvh1WlA",
      "eyJraWQiOiJkaWQ6ZXhhbXBsZ...KPxgihac0aW9EkL1nOzM"
    ]
  }
}
```

The Client MAY request encrypted responses by providing its encryption parameters in the Credential Request.

The Credential Issuer indicates support for encrypted responses by including the `credential_response_encryption` parameter in the Credential Issuer Metadata.

### Proof Types {#proof-types}

This specification defines the following proof types:

* `jwt`: A JWT [@!RFC7519] is used for proof of possession. When a `proof_type` parameter in a `proof` object is set to `jwt`, it MUST also contain a `jwt` parameter that includes a JWT as defined in (#jwt-proof-type). When a `proofs` object is using a `jwt` proof type, it MUST include a `jwt` parameter with its value being an array of JWTs, where each JWT is formed as defined in (#jwt-proof-type).
* `ldp_vp`: A W3C Verifiable Presentation object signed using the Data Integrity Proof [@VC_Data_Integrity] as defined in [@VC_DATA_2.0] or [@VC_DATA] is used for proof of possession. When a `proof_type` parameter in a `proof` object is set to `ldp_vp`, it MUST also contain an `ldp_vp` parameter that includes a [W3C Verifiable Presentation](https://www.w3.org/TR/vc-data-model-2.0/#presentations-0) defined in (#ldp-vp-proof-type). When a `proofs` object is using a `ldp_vp` proof type, it MUST include an `ldp_vp` parameter with its value being an array of [W3C Verifiable Presentations](https://www.w3.org/TR/vc-data-model-2.0/#presentations-0), where each of these W3C Verifiable Presentation is formed as defined in (#ldp-vp-proof-type).

Additional proof types MAY be defined and used.

#### `jwt` Proof Type {#jwt-proof-type}

The JWT MUST contain the following elements:

* in the JOSE header,
  * `alg`: REQUIRED. A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry [@IANA.JOSE.ALGS]. It MUST NOT be `none` or an identifier for a symmetric algorithm (MAC).
  * `typ`: REQUIRED. MUST be `openid4vci-proof+jwt`, which explicitly types the key proof JWT as recommended in Section 3.11 of [@!RFC8725].
  * `kid`: OPTIONAL. JOSE Header containing the key ID. If the Credential shall be bound to a DID, the `kid` refers to a DID URL which identifies a particular key in the DID Document that the Credential shall be bound to. It MUST NOT be present if `jwk` is present.
  * `jwk`: OPTIONAL. JOSE Header containing the key material the new Credential shall be bound to. It MUST NOT be present if `kid` is present.
  * `x5c`: OPTIONAL. JOSE Header containing a certificate or certificate chain corresponding to the key used to sign the JWT. This element MAY be used to convey a key attestation. In such a case, the actual key certificate will contain attributes related to the key properties.
  * `trust_chain`: OPTIONAL. JOSE Header containing an [@!OpenID.Federation] Trust Chain. This element MAY be used to convey key attestation, metadata, metadata policies, federation Trust Marks and any other information related to a specific federation, if available in the chain. When used for signature verification, the header parameter `kid` MUST be present.

* in the JWT body,
  * `iss`: OPTIONAL (string). The value of this claim MUST be the `client_id` of the Client making the Credential request. This claim MUST be omitted if the access token authorizing the issuance call was obtained from a Pre-Authorized Code Flow through anonymous access to the token endpoint.
  * `aud`: REQUIRED (string). The value of this claim MUST be the Credential Issuer Identifier.
  * `iat`: REQUIRED (number). The value of this claim MUST be the time at which the key proof was issued using the syntax defined in [@!RFC7519].
  * `nonce`: OPTIONAL (string). The value type of this claim MUST be a string, where the value is a server-provided `c_nonce`. It MUST be present when the Wallet received a server-provided `c_nonce`.

The Credential Issuer MUST validate that the JWT used as a proof is actually signed by a key identified in the JOSE Header.

Cryptographic algorithm names used in the `proof_signing_alg_values_supported` Credential Issuer metadata parameter for this proof type SHOULD be one of those defined in [@IANA.JOSE.ALGS].

Below is a non-normative example of a `proof` parameter (with line breaks within values for display purposes only):

```json
{
  "proof_type": "jwt",
  "jwt": 
  "eyJ0eXAiOiJvcGVuaWQ0dmNpLXByb29mK2p3dCIsImFsZyI6IkVTMjU2IiwiandrI
  jp7Imt0eSI6IkVDIiwiY3J2IjoiUC0yNTYiLCJ4IjoiblVXQW9BdjNYWml0aDhFN2k
  xOU9kYXhPTFlGT3dNLVoyRXVNMDJUaXJUNCIsInkiOiJIc2tIVThCalVpMVU5WHFpN
  1N3bWo4Z3dBS18weGtjRGpFV183MVNvc0VZIn19.eyJhdWQiOiJodHRwczovL2NyZW
  RlbnRpYWwtaXNzdWVyLmV4YW1wbGUuY29tIiwiaWF0IjoxNzAxOTYwNDQ0LCJub25j
  ZSI6IkxhclJHU2JtVVBZdFJZTzZCUTR5bjgifQ.-a3EDsxClUB4O3LeDD5DVGEnNMT
  01FCQW4P6-2-BNBqc_Zxf0Qw4CWayLEpqkAomlkLb9zioZoipdP-jvh1WlA"
}
```

where the decoded JWT looks like this:

```json
{
  "typ": "openid4vci-proof+jwt",
  "alg": "ES256",
  "jwk": {
    "kty": "EC",
    "crv": "P-256",
    "x": "nUWAoAv3XZith8E7i19OdaxOLYFOwM-Z2EuM02TirT4",
    "y": "HskHU8BjUi1U9Xqi7Swmj8gwAK_0xkcDjEW_71SosEY"
  }
}.{
  "aud": "https://credential-issuer.example.com",
  "iat": 1701960444,
  "nonce": "LarRGSbmUPYtRYO6BQ4yn8"
}
```

Here is another example JWT not only proving possession of a private key but also providing key attestation data for that key:

```json
{
  "alg": "ES256",
  "x5c": [<key certificate + certificate chain for attestation>]
}.
{
  "iss": "s6BhdRkqt3",
  "aud": "https://server.example.com",
  "iat": 1659145924,
  "nonce": "tZignsnFbp"
}
```

#### `ldp_vp` Proof Type {#ldp-vp-proof-type}

When a W3C Verifiable Presentation as defined by [@VC_DATA_2.0] or [@VC_DATA] signed using Data Integrity is used as key proof, it MUST contain the following elements:

* `holder`: OPTIONAL. MUST be equivalent to the controller identifier (e.g., DID) for the `verificationMethod` value identified by the `proof.verificationMethod` property.
* `proof`: REQUIRED. The proof body of a W3C Verifiable Presentation.
  * `domain`: REQUIRED (string). The value of this claim MUST be the Credential Issuer Identifier.
  * `challenge`: REQUIRED when the Credential Issuer has provided a `c_nonce`. It MUST NOT be used otherwise. String, where the value is a server-provided `c_nonce`. It MUST be present when the Wallet received a server-provided `c_nonce`.

The Credential Issuer MUST validate that the W3C Verifiable Presentation used as a proof is actually signed with a key in the possession of the Holder.

Cryptographic algorithm names used in the `proof_signing_alg_values_supported` Credential Issuer metadata parameter for this proof type SHOULD be one of those defined in [@LD_Suite_Registry].

Below is a non-normative example of a `proof` parameter:

```json
{
  "proof_type": "ldp_vp",
  "ldp_vp": {
    "@context": [
      "https://www.w3.org/ns/credentials/v2",
      "https://www.w3.org/ns/credentials/examples/v2"
    ],
    "type": [
      "VerifiablePresentation"
    ],
    "holder": "did:key:z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mqh5XPvLsrPsro",
    "proof": [
      {
        "type": "DataIntegrityProof",
        "cryptosuite": "eddsa-2022",
        "proofPurpose": "authentication",
        "verificationMethod": "did:key:z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mqh5XPvLsrPsro#z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mqh5XPvLsrPsro",
        "created": "2023-03-01T14:56:29.280619Z",
        "challenge": "82d4cb36-11f6-4273-b9c6-df1ac0ff17e9",
        "domain": "did:web:audience.company.com",
        "proofValue": "z5hrbHzZiqXHNpLq6i7zePEUcUzEbZKmWfNQzXcUXUrqF7bykQ7ACiWFyZdT2HcptF1zd1t7NhfQSdqrbPEjZceg7"
      }
    ]
  }
}

```

### Verifying Proof {#verifying-key-proof}

To validate a key proof, the Credential Issuer MUST ensure that:

- all required claims for that proof type are contained as defined in (#proof-types),
- the key proof is explicitly typed using header parameters as defined for that proof type,
- the header parameter indicates a registered asymmetric digital signature algorithm, `alg` parameter value is not `none`, is supported by the application, and is acceptable per local policy,
- the signature on the key proof verifies with the public key contained in the header parameter,
- the header parameter does not contain a private key,
- if the server had previously provided a `c_nonce`, the nonce in the key proof matches the server-provided `c_nonce` value,
- the creation time of the JWT, as determined by either the issuance time, or a server managed timestamp via the nonce claim, is within an acceptable window (see (#key-proof-replay)).

These checks may be performed in any order.

## Credential Response {#credential-response}

Credential Response can contain one or more Credentials depending on the Credential Request.

Credential Response can be immediate or deferred and can contain one or more Credentials with the same Credential Configuration and Credential Dataset depending on the Credential Request. The Credential Issuer MAY be able to immediately issue requested Credentials and send them to the Client. In other cases, the Credential Issuer MAY NOT be able to immediately issue a requested Credential and would send a `transaction_id` parameter to the Client to be used later to receive a Credential when it is ready.

The HTTP status code MUST be 202 (see Section 15.3.3 of [@!RFC9110]).

If the Client requested an encrypted response by including the `credential_response_encryption` object in the request, the Credential Issuer MUST encode the information in the Credential Response as a JWT using the  parameters from the `credential_response_encryption` object. If the Credential Response is encrypted, the media type of the response MUST be set to `application/jwt`. If encryption was requested in the Credential Request and the Credential Response is not encrypted, the Client SHOULD reject the Credential Response.

If the Credential Response is not encrypted, the media type of the response MUST be set to `application/json`.

The following parameters are used in the JSON-encoded Credential Response body:

* `credentials`: OPTIONAL. Contains an array of one or more issued Credentials. It MUST NOT be used if the `transaction_id` parameter is present. The elements of the array MUST be objects. This specification defines the following parameters to be used inside this object:
   * `credential`: REQUIRED. Contains one issued Credential. It MAY be a string or an object, depending on the Credential Format. See Appendix A for the Credential Format-specific encoding requirements.
* `transaction_id`: OPTIONAL. String identifying a Deferred Issuance transaction. This parameter is contained in the response if the Credential Issuer cannot immediately issue the Credential. The value is subsequently used to obtain the respective Credential with the Deferred Credential Endpoint (see (#deferred-credential-issuance)). It MUST not be used if the `credentials` parameter is present. It MUST be invalidated after the Credential for which it was meant has been obtained by the Wallet.
* `notification_id`: OPTIONAL. String identifying one or more Credentials issued in one Credential Response. It MUST be included in the Notification Request as defined in (#notification). It MUST not be used if the `credentials` parameter is not present.

The format of the Credential in the Credential Response is determined by the value of the `format` parameter specified in the Credential Request.

The encoding of the Credential returned in the `credential` parameter depends on the Credential Format. Credential Formats expressed as binary data MUST be base64url-encoded and returned as a string.

More details such as Credential Format Identifiers are defined in the Credential Format Profiles in (#format-profiles). 

Additional Credential Response parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

Below is a non-normative example of a Credential Response in an immediate issuance flow for a Credential in JWT VC format (JSON encoded):

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "credentials": [
    {
      "credential": "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L"
    }
  ]
}
```

Below is a non-normative example of a Credential Response in an immediate issuance flow for multiple Credential instances in JWT VC format (JSON encoded) with an additional `notification_id` parameter:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "credentials": [
    {
      "credential": "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L"
    },
    {
      "credential": "YXNkZnNhZGZkamZqZGFza23....29tZTIzMjMyMzIzMjMy"
    }
  ],
  "notification_id": "3fwe98js"
}
```

Below is a non-normative example of a Credential Response in a deferred flow:

```
HTTP/1.1 202 Accepted
Content-Type: application/json
Cache-Control: no-store

{
  "transaction_id": "8xLOxBtZp8"
}
```

### Credential Error Response {#credential-error-response}

When the Credential Request is invalid or unauthorized, the Credential Issuer constructs the error response as defined in this section.

#### Authorization Errors {#authorization-errors}

If the Credential Request does not contain an Access Token that enables issuance of a requested Credential, the Credential Endpoint returns an authorization error response such as defined in Section 3 of [@!RFC6750].

#### Credential Request Errors {#credential-request-errors}

For errors related to the Credential Request's payload, such as issues with `type`, `format`, `proof`, encryption parameters, or if the request is denied, the specific error codes from this section MUST be used instead of the generic `invalid_request` parameter defined in Section 3.1 of [@!RFC6750].

If the Wallet is requesting the issuance of a Credential that is not supported by the Credential Endpoint, the HTTP response MUST use the HTTP status code 400 (Bad Request) and set the content type to `application/json` with the following parameters in the JSON-encoded response body:

* `error`: REQUIRED. The `error` parameter SHOULD be a single ASCII [@!USASCII] error code from the following:
  * `invalid_credential_request`: The Credential Request is missing a required parameter, includes an unsupported parameter or parameter value, repeats the same parameter, or is otherwise malformed.
  * `unsupported_credential_type`: Requested Credential type is not supported.
  * `unsupported_credential_format`: Requested Credential Format is not supported.
  * `invalid_proof`: The `proof` or `proofs` parameter in the Credential Request is invalid: (1) if both fields are missing, or (2) both are present simultaneously, or (3) one of the provided key proofs is invalid, or (4) if at least one of the key proofs is not linked to a required `c_nonce` value (refer to (#nonce-response)).
  * `invalid_nonce`: The `proof` or `proofs` parameter in the Credential Request uses an invalid nonce: at least one of the key proofs is not linked to a valid `c_nonce` value. The wallet should retrieve a new `c_nonce` value (refer to (#nonce-endpoint)).
  * `invalid_encryption_parameters`: This error occurs when the encryption parameters in the Credential Request are either invalid or missing. In the latter case, it indicates that the Credential Issuer requires the Credential Response to be sent encrypted, but the Credential Request does not contain the necessary encryption parameters.
  * `credential_request_denied`: The Credential Request has not been accepted by the Credential Issuer.
* `error_description`: OPTIONAL. The `error_description` parameter MUST be a human-readable ASCII [@!USASCII] text, providing any additional information used to assist the Client implementers in understanding the occurred error. The values for the `error_description` parameter MUST NOT include characters outside the set `%x20-21 / %x23-5B / %x5D-7E`.

The usage of these parameters takes precedence over the `invalid_request` parameter defined in (#authorization-errors), since they provide more details about the errors.

The following is a non-normative example of a Credential Error Response where an unsupported Credential Format was requested:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "unsupported_credential_format"
}
```

# Deferred Credential Endpoint {#deferred-credential-issuance}

This endpoint is used to issue one or more Credentials previously requested at the Credential Endpoint in cases where the Credential Issuer was not able to immediately issue this Credential. Support for this endpoint is OPTIONAL.

The Wallet MUST present to the Deferred Endpoint an Access Token that is valid for the issuance of the Credential(s) previously requested at the Credential Endpoint. 

Communication with the Deferred Credential Endpoint MUST utilize TLS. 

## Deferred Credential Request {#deferred-credential-request}

The Deferred Credential Request is an HTTP POST request. It MUST be sent using the `application/json` media type.

The following parameter is used in the Deferred Credential Request:

* `transaction_id`: REQUIRED. String identifying a Deferred Issuance transaction.

The Credential Issuer MUST invalidate the `transaction_id` after the Credential for which it was meant has been obtained by the Wallet.

Additional Deferred Credential Request parameters MAY be defined and used.
The Credential Issuer MUST ignore any unrecognized parameters.

The following is a non-normative example of a Deferred Credential Request:

```
POST /deferred_credential HTTP/1.1 
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "transaction_id": "8xLOxBtZp8"
}
```

## Deferred Credential Response {#deferred-credential-response}

The Deferred Credential Response uses the `credentials` and `notification_id` parameters as defined in (#credential-response).

Additional Deferred Credential Response parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

The Deferred Credential Response MUST be sent using the `application/json` media type.

The following is a non-normative example of a Deferred Credential Response:

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "credentials": [
    {
      "credential": "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L"
    },
    {
      "credential": "YXNkZnNhZGZkamZqZGFza23....29tZTIzMjMyMzIzMjMy"
    }
  ],
  "notification_id": "3fwe98js"
}
```

## Deferred Credential Error Response {#deferred-credential-error-response}

When the Deferred Credential Request is invalid or the Credential is not available yet, the Credential Issuer constructs the error response as defined in (#credential-error-response).

The following additional error codes are specified in addition to those already defined in (#credential-request-errors):

* `issuance_pending`: The Credential issuance is still pending. The error response SHOULD also contain the `interval` member, determining the minimum amount of time in seconds that the Wallet needs to wait before providing a new request to the Deferred Credential Endpoint.  If `interval` member is not present, the Wallet MUST use `5` as the default value.
* `invalid_transaction_id`: The Deferred Credential Request contains an invalid `transaction_id`. This error occurs when the `transaction_id` was not issued by the respective Credential Issuer or it was already used to obtain a Credential.

This is a non-normative example of a Deferred Credential Error Response:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "invalid_transaction_id"
}
```

# Notification Endpoint {#notification-endpoint}

This endpoint is used by the Wallet to notify the Credential Issuer of certain events for issued Credentials. These events enable the Credential Issuer to take subsequent actions after issuance. The Credential Issuer needs to return one `notification_id` parameter per Credential Response or Deferred Credential Response for the Wallet to be able to use this endpoint. Support for this endpoint is OPTIONAL. The Issuer cannot assume that a notification will be sent for every issued Credential since the use of this Endpoint is not mandatory for the Wallet.

The Wallet MUST present to the Notification Endpoint a valid Access Token issued at the Token Endpoint as defined in (#token-endpoint). 

A Credential Issuer that requires a request to the Notification Endpoint MUST ensure the Access Token issued by the Authorization Server is valid at the Notification Endpoint.

The notification from the Wallet is idempotent. When the Credential Issuer receives multiple identical calls from the Wallet for the same `notification_id`, it returns success. Due to the network errors, there are no guarantees that a Credential Issuer will receive a notification within a certain time period or at all.

Communication with the Notification Endpoint MUST utilize TLS.

## Notification Request {#notification}

The Wallet sends an HTTP POST request to the Notification Endpoint with the following parameters in the entity-body and using the `application/json` media type. If the Wallet supports the Notification Endpoint, the Wallet MAY send one or more Notification Requests per `notification_id` value received.
* `notification_id`: REQUIRED. String received in the Credential Response or Deferred Credential Response identifying an issuance flow that contained one or more Credentials with the same Credential Configuration and Credential Dataset.
* `event`: REQUIRED. Type of the notification event. It MUST be a case sensitive string whose value is either `credential_accepted`, `credential_failure`, or `credential_deleted`. `credential_accepted` is to be used when the Credentials were successfully stored in the Wallet, with or without user action. `credential_deleted` is to be used when the unsuccessful Credential issuance was caused by a user action. In all other unsuccessful cases, `credential_failure` is to be used. Partial errors during batch credential issuance (e.g., one of the Credentials in the batch could not be stored) MUST be treated as the overall issuance flow failing.

* `event_description`: OPTIONAL. Human-readable ASCII [@!USASCII] text providing additional information, used to assist the Credential Issuer developer in understanding the event that occurred. Values for the `event_description` parameter MUST NOT include characters outside the set `%x20-21 / %x23-5B / %x5D-7E`.

Additional Notification Request parameters MAY be defined and used.
The Credential Issuer MUST ignore any unrecognized parameters.

Below is a non-normative example of a Notification Request when a credential was successfully accepted by the End-User:

```
POST /notification HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "notification_id": "3fwe98js",
  "event": "credential_accepted"
}
```

Below is a non-normative example of a Notification Request when a Credential was deleted by the End-User:

```
POST /notification HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "notification_id": "3fwe98js",
  "event": "credential_failure",
  "event_description": "Could not store the Credential. Out of storage."
}
```

## Successful Notification Response

When the Credential Issuer has successfully received the Notification Request from the Wallet, it MUST respond with an HTTP status code in the 2xx range. Use of the HTTP status code 204 (No Content) is RECOMMENDED.

Below is a non-normative example of response to a successful Notification Request:

```
HTTP/1.1 204 No Content
```

## Notification Error Response

If the Notification Request does not contain an Access Token or contains an invalid Access Token, the Notification Endpoint returns an Authorization Error Response, such as defined in Section 3 of [@!RFC6750].

When the `notification_id` value is invalid, the HTTP response MUST use the HTTP status code 400 (Bad Request) and set the content type to `application/json` with the following parameters in the JSON-encoded response body:

* `error`: REQUIRED. The value of the `error` parameter SHOULD be one of the following ASCII [@!USASCII] error codes:
  * `invalid_notification_id`: The `notification_id` in the Notification Request was invalid.
  * `invalid_notification_request`: The Notification Request is missing a required parameter, includes an unsupported parameter or parameter value, repeats the same parameter, or is otherwise malformed.

It is at the discretion of the Issuer to decide how to proceed after returning an error response.

The following is a non-normative example of a Notification Error Response when an invalid `notification_id` value was used:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store
{
  "error": "invalid_notification_id"
}
```

# Metadata

## Client Metadata {#client-metadata}

This specification defines the following new Client Metadata parameter in addition to those defined by [@!RFC7591] for Wallets acting as OAuth 2.0 Client:

* `credential_offer_endpoint`: OPTIONAL. Credential Offer Endpoint of a Wallet.

Additional Client metadata parameters MAY be defined and used,
as described in [@!RFC7591].
The Wallet MUST ignore any unrecognized parameters.

### Client Metadata Retrieval {#client-metadata-retrieval}

How to obtain Client Metadata is out of scope of this specification. Profiles of this specification MAY also define static sets of Client Metadata values to be used.

If the Credential Issuer is unable to perform discovery of the Wallet's Credential Offer Endpoint, the following custom URL scheme is used: `openid-credential-offer://`.

## Credential Issuer Metadata {#credential-issuer-metadata}

The Credential Issuer Metadata contains information on the Credential Issuer's technical capabilities, supported Credentials, and (internationalized) display information.

### Credential Issuer Identifier {#credential-issuer-identifier}

A Credential Issuer is identified by a case sensitive URL using the `https` scheme that contains scheme, host and, optionally, port number and path components, but no query or fragment components. 

### Credential Issuer Metadata Retrieval {#credential-issuer-wellknown}

The Credential Issuer's configuration can be retrieved using the Credential Issuer Identifier.

Credential Issuers publishing metadata MUST make a JSON document available at the path formed by concatenating the string `/.well-known/openid-credential-issuer` to the Credential Issuer Identifier. If the Credential Issuer value contains a path component, any terminating `/` MUST be removed before appending `/.well-known/openid-credential-issuer`.

Communication with the Credential Issuer Metadata Endpoint MUST utilize TLS.

To fetch the Credential Issuer Metadata, the requester MUST send an HTTP request using the GET method and the path formed following the steps above. The Credential Issuer MUST return a JSON document compliant with this specification using the `application/json` media type and the HTTP Status Code 200.

The Wallet is RECOMMENDED to send an `Accept-Language` Header in the HTTP GET request to indicate the language(s) preferred for display. It is up to the Credential Issuer whether to:

* send a subset the metadata containing internationalized display data for one or all of the requested languages and indicate returned languages using the HTTP `Content-Language` Header, or
* ignore the `Accept-Language` Header and send all supported languages or any chosen subset.

The language(s) in HTTP `Accept-Language` and `Content-Language` Headers MUST use the values defined in [@!RFC3066]. 

Below is a non-normative example of a Credential Issuer Metadata request:

```
GET /.well-known/openid-credential-issuer HTTP/1.1
Host: server.example.com
Accept-Language: fr-ch, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5
```

### Credential Issuer Metadata Parameters {#credential-issuer-parameters}

This specification defines the following Credential Issuer Metadata parameters:

* `credential_issuer`: REQUIRED. The Credential Issuer's identifier, as defined in (#credential-issuer-identifier).
* `authorization_servers`: OPTIONAL. Array of strings, where each string is an identifier of the OAuth 2.0 Authorization Server (as defined in [@!RFC8414]) the Credential Issuer relies on for authorization. If this parameter is omitted, the entity providing the Credential Issuer is also acting as the Authorization Server, i.e., the Credential Issuer's identifier is used to obtain the Authorization Server metadata. The actual OAuth 2.0 Authorization Server metadata is obtained from the `oauth-authorization-server` well-known location as defined in Section 3 of [@!RFC8414]. When there are multiple entries in the array, the Wallet may be able to determine which Authorization Server to use by querying the metadata; for example, by examining the `grant_types_supported` values, the Wallet can filter the server to use based on the grant type it plans to use. When the Wallet is using `authorization_server` parameter in the Credential Offer as a hint to determine which Authorization Server to use out of multiple, the Wallet MUST NOT proceed with the flow if the `authorization_server` Credential Offer parameter value does not match any of the entries in the `authorization_servers` array.
* `credential_endpoint`: REQUIRED. URL of the Credential Issuer's Credential Endpoint, as defined in (#credential-request). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components.
* `nonce_endpoint`: OPTIONAL. URL of the Credential Issuer's Nonce Endpoint, as defined in (#nonce-endpoint). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Nonce Endpoint.
* `deferred_credential_endpoint`: OPTIONAL. URL of the Credential Issuer's Deferred Credential Endpoint, as defined in (#deferred-credential-issuance). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Deferred Credential Endpoint.
* `notification_endpoint`: OPTIONAL. URL of the Credential Issuer's Notification Endpoint, as defined in (#notification-endpoint). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Notification Endpoint.
* `credential_response_encryption`: OPTIONAL. Object containing information about whether the Credential Issuer supports encryption of the Credential Credential Response on top of TLS.
  * `alg_values_supported`: REQUIRED. Array containing a list of the JWE [@!RFC7516] encryption algorithms (`alg` values) [@!RFC7518] supported by the Credential Endpoint to encode the Credential Response in a JWT [@!RFC7519].
  * `enc_values_supported`: REQUIRED. Array containing a list of the JWE [@!RFC7516] encryption algorithms (`enc` values) [@!RFC7518] supported by the Credential Endpoint to encode the Credential Response in a JWT [@!RFC7519].
  * `encryption_required`: REQUIRED. Boolean value specifying whether the Credential Issuer requires the additional encryption on top of TLS for the Credential Response. If the value is `true`, the Credential Issuer requires encryption for every Credential Response and therefore the Wallet MUST provide encryption keys in the Credential Request. If the value is `false`, the Wallet MAY chose whether it provides encryption keys or not.
* `batch_credential_issuance`: OPTIONAL. Object containing information about the Credential Issuer's supports for batch issuance of Credentials on the Credential Endpoint. The presence of this parameter means that the issuer supports the `proofs` parameter in the Credential Request so can issue more than one Verifiable Credential for the same Credential Dataset in a single request/response.
  * `batch_size`: REQUIRED. Integer value specifying the maximum array size for the `proofs` parameter in a Credential Request.
* `signed_metadata`: OPTIONAL. String that is a signed JWT. This JWT contains Credential Issuer metadata parameters as claims. The signed metadata MUST be secured using JSON Web Signature (JWS) [@!RFC7515] and MUST contain an `iat` (Issued At) claim, an `iss` (Issuer) claim denoting the party attesting to the claims in the signed metadata, and `sub` (Subject) claim matching the Credential Issuer identifier. If the Wallet supports signed metadata, metadata values conveyed in the signed JWT MUST take precedence over the corresponding values conveyed using plain JSON elements. If the Credential Issuer wants to enforce use of signed metadata, it omits the respective metadata parameters from the unsigned part of the Credential Issuer metadata. A `signed_metadata` metadata value MUST NOT appear as a claim in the JWT. The Wallet MUST establish trust in the signer of the metadata, and obtain the keys to validate the signature before processing the metadata. The concrete mechanism how to do that is out of scope of this specification and MAY be defined in the profiles of this specification.
* `display`: OPTIONAL. Array of objects, where each object contains display properties of a Credential Issuer for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
  * `name`: OPTIONAL. String value of a display name for the Credential Issuer.
  * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
  * `logo`: OPTIONAL. Object with information about the logo of the Credential Issuer. Below is a non-exhaustive list of parameters that MAY be included:
    * `uri`: REQUIRED. String value that contains a URI where the Wallet can obtain the logo of the Credential Issuer. The Wallet needs to determine the scheme, since the URI value could use the `https:` scheme, the `data:` scheme, etc.
    * `alt_text`: OPTIONAL. String value of the alternative text for the logo image.
* `credential_configurations_supported`: REQUIRED. Object that describes specifics of the Credential that the Credential Issuer supports issuance of. This object contains a list of name/value pairs, where each name is a unique identifier of the supported Credential being described. This identifier is used in the Credential Offer as defined in (#credential-offer-parameters) to communicate to the Wallet which Credential is being offered. The value is an object that contains metadata about a specific Credential and contains the following parameters defined by this specification:
  * `format`: REQUIRED. A JSON string identifying the format of this Credential, i.e., `jwt_vc_json` or `ldp_vc`. Depending on the format value, the object contains further elements defining the type and (optionally) particular claims the Credential MAY contain and information about how to display the Credential. (#format-profiles) contains Credential Format Profiles introduced by this specification.
  * `scope`: OPTIONAL. A JSON string identifying the scope value that this Credential Issuer supports for this particular Credential. The value can be the same across multiple `credential_configurations_supported` objects. The Authorization Server MUST be able to uniquely identify the Credential Issuer based on the scope value. The Wallet can use this value in the Authorization Request as defined in (#credential-request-using-type-specific-scope). Scope values in this Credential Issuer metadata MAY duplicate those in the `scopes_supported` parameter of the Authorization Server.
  * `cryptographic_binding_methods_supported`: OPTIONAL. Array of case sensitive strings that identify the representation of the cryptographic key material that the issued Credential is bound to, as defined in (#credential-binding). Support for keys in JWK format [@!RFC7517] is indicated by the value `jwk`. Support for keys expressed as a COSE Key object [@!RFC8152] (for example, used in [@!ISO.18013-5]) is indicated by the value `cose_key`. When the Cryptographic Binding Method is a DID, valid values are a `did:` prefix followed by a method-name using a syntax as defined in Section 3.1 of [@!DID-Core], but without a `:`and method-specific-id. For example, support for the DID method with a method-name "example" would be represented by `did:example`.
  * `credential_signing_alg_values_supported`: OPTIONAL. Array of case sensitive strings that identify the algorithms that the Issuer uses to sign the issued Credential. Algorithm names used are determined by the Credential Format and are defined in (#format-profiles).
  * `proof_types_supported`: OPTIONAL. Object that describes specifics of the key proof(s) that the Credential Issuer supports. This object contains a list of name/value pairs, where each name is a unique identifier of the supported proof type(s). Valid values are defined in (#proof-types), other values MAY be used. This identifier is also used by the Wallet in the Credential Request as defined in (#credential-request). The value in the name/value pair is an object that contains metadata about the key proof and contains the following parameters defined by this specification:
    * `proof_signing_alg_values_supported`: REQUIRED. Array of case sensitive strings that identify the algorithms that the Issuer supports for this proof type. The Wallet uses one of them to sign the proof. Algorithm names used are determined by the key proof type and are defined in (#proof-types).
  * `display`: OPTIONAL. Array of objects, where each object contains the display properties of the supported Credential for a certain language. Below is a non-exhaustive list of parameters that MAY be included.
      * `name`: REQUIRED. String value of a display name for the Credential.
      * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. Multiple `display` objects MAY be included for separate languages. There MUST be only one object for each language identifier.
      * `logo`: OPTIONAL. Object with information about the logo of the Credential. The following non-exhaustive set of parameters MAY be included:
          * `uri`: REQUIRED. String value that contains a URI where the Wallet can obtain the logo of the Credential from the Credential Issuer. The Wallet needs to determine the scheme, since the URI value could use the `https:` scheme, the `data:` scheme, etc.
          * `alt_text`: OPTIONAL. String value of the alternative text for the logo image.
      * `description`: OPTIONAL. String value of a description of the Credential.
      * `background_color`: OPTIONAL. String value of a background color of the Credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].
      * `background_image`: OPTIONAL. Object with information about the background image of the Credential. At least the following parameter MUST be included:
          * `uri`: REQUIRED. String value that contains a URI where the Wallet can obtain the background image of the Credential from the Credential Issuer. The Wallet needs to determine the scheme, since the URI value could use the `https:` scheme, the `data:` scheme, etc.
      * `text_color`: OPTIONAL. String value of a text color of the Credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].

An Authorization Server that only supports the Pre-Authorized Code grant type MAY omit the `response_types_supported` parameter in its metadata despite [@!RFC8414] mandating it.

Note: It can be challenging for a Credential Issuer that accepts tokens from multiple Authorization Servers to introspect an Access Token to check the validity and determine the permissions granted. Some ways to achieve this are relying on Authorization Servers that use [@!RFC9068] or by the Credential Issuer understanding the proprietary Access Token structures of the Authorization Servers.

Depending on the Credential Format, additional parameters might be present in the `credential_configurations_supported` object values, such as information about claims in the Credential. For Credential Format-specific claims, see the "Credential Issuer Metadata" subsections in (#format-profiles).

The Authorization Server MUST be able to determine from the Issuer metadata what claims are disclosed by the requested Credentials to be able to render meaningful End-User consent.

Additional Credential Issuer metadata parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

The following is a non-normative example of Credential Issuer metadata of a Credential in the IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format:

<{{examples/credential_metadata_sd_jwt_vc.json}}

Note: The Client MAY use other mechanisms to obtain information about the Verifiable Credentials that a Credential Issuer can issue.

## OAuth 2.0 Authorization Server Metadata

This specification also defines a new OAuth 2.0 Authorization Server metadata [@!RFC8414] parameter to publish whether the Authorization Server that the Credential Issuer relies on for authorization supports anonymous Token Requests with the Pre-Authorized Grant Type. It is defined as follows:

* `pre-authorized_grant_anonymous_access_supported`: OPTIONAL. A boolean indicating whether the Credential Issuer accepts a Token Request with a Pre-Authorized Code but without a `client_id`. The default is `false`. 

Additional Authorization Server metadata parameters MAY be defined and used,
as described in [@!RFC8414].
The Wallet MUST ignore any unrecognized parameters.

# Security Considerations {#security-considerations}

## Trust between Wallet and Issuer

Credential Issuers often want to know what Wallet they are issuing Credentials to and how private keys are managed for the following reasons:

* The Credential Issuer MAY want to ensure that private keys are properly protected from exfiltration and replay to prevent an adversary from impersonating the legitimate Credential Holder by presenting their Credentials.
* The Credential Issuer MAY also want to ensure that the Wallet managing the Credentials adheres to certain policies and, potentially, was audited and approved under a certain regulatory and/or commercial scheme.

The following mechanisms in concert can be utilized to fulfill those objectives:

**Key attestation** is a mechanism where the device or security element in a device asserts the key management policy to the application creating and using this key. The Android Operating System, for example, provides apps with a certificate including a certificate chain asserting that a particular key is managed, for example, by a hardware security module. The Wallet can provide this data along with the proof of possession in the Credential Request (see (#credential-request) for an example) to allow the Credential Issuer to validate the key management policy. This requires the Credential Issuer to rely on the trust anchor of the certificate chain and the respective key management policy. Another variant of this concept is the use of a Qualified Electronic Signature as defined by the eIDAS regulation [@eIDAS]. This signature will not reveal the properties of the associated private key to the Credential Issuer. However, as one example, due to the regulatory regime of eIDAS, the Credential Issuer can deduce that the signing service manages the private keys according to this regime and fulfills very high security requirements. As another example, FIDO2 allows RPs to obtain an attestation along with the public key from a FIDO authenticator. That implicitly asserts the key management policy, since the assertion is bound to a certain authenticator model and its key management capabilities.

**App Attestation**: Key attestation, however, does not establish trust in the application storing the Credential and producing presentation of that Credential. App attestation, as provided by mobile operating systems, e.g., iOS's DeviceCheck or Android's SafetyNet, allows a server system to ensure it is communicating to a legitimate instance of its genuine app. Those mechanisms can be utilized to validate the internal integrity of the Wallet (as a whole).

**Device Attestation**: Device Attestation attests to the health of the device on which the Wallet is running, as a whole. It helps prevent compromises such as a malicious third-party application tampering with a Wallet that manages keys and Credentials, which cannot be captured only by obtaining app attestation of a Wallet.

**Client Authentication** allows a Wallet to authenticate with a Credential Issuer. To securely authenticate, the Wallet needs to utilize a backend component managing the key material and processing the secure communication with the Credential Issuer. The Credential Issuer MAY establish trust with the Wallet based on its own auditing or a trusted third-party attestation of the conformance of the Wallet to a certain policy.

Directly using key, app, and/or device attestations to prove certain capabilities to a Credential Issuer is an obvious option. However, this at least requires platform mechanisms that issue signed assertions that third parties can evaluate, which is not always the case (e.g., iOS's DeviceCheck). Also, such an approach creates dependencies on platform specific mechanisms, trust anchors, and platform specific identifiers (e.g., Android `apkPackageName`) and it reveals information about the internal design of a Wallet. Implementers should take these consequences into account.

The approach recommended by this specification is that the Credential Issuer relies on the OAuth 2.0 Client Authentication to establish trust in the Wallet and leaves it to the Wallet to ensure its internal integrity using app and key attestation (if required). This establishes a clean separation between the different components and a uniform interface irrespective of the Wallet's architecture (e.g., native vs. Web Wallet). Client Authentication can be performed with assertions registered with the Credential Issuer or with assertions issued to the Wallet by a third party the Credential Issuer trusts for the purpose of Client Authentication.

## Credential Offer {#credential-offer-security}

The Wallet MUST consider the parameter values in the Credential Offer as not trustworthy, since the origin is not authenticated and the message integrity is not protected. The Wallet MUST apply the same checks on the Credential Issuer that it would apply when the flow is started from the Wallet itself, since the Credential Issuer is not trustworthy just because it sent the Credential Offer. An attacker might attempt to use a Credential Offer to conduct a phishing or injection attack.

The Wallet MUST NOT accept Credentials just because this mechanism was used. All protocol steps defined in this specification MUST be performed in the same way as if the Wallet would have started the flow. 

The Credential Issuer MUST ensure the release of any privacy-sensitive data in Credential Offer is legal.

## Pre-Authorized Code Flow {#security-considerations-pre-authz-code}

### Replay Prevention

The Pre-Authorized Code Flow is vulnerable to the replay of the Pre-Authorized Code, because by design, it is not bound to a certain device (as the Authorization Code Flow does with PKCE). This means an attacker can replay the Pre-Authorized Code meant for a victim at another device, e.g., the attacker can scan the QR code while it is displayed on the victim's screen, and thereby get access to the Credential. Such replay attacks must be prevented using other means. The design facilitates the following options:

* Transaction Code: the Credential Issuer might set up a Transaction Code with the End-User (e.g., via text message or email) that needs to be presented in the Token Request.

### Transaction Code Phishing

An attacker might leverage the Credential issuance process and the End-User's trust in the Wallet to phish Transaction Codes sent out by a different service that grant the attacker access to services other than Credential issuance. The attacker could set up a Credential Issuer site and in parallel to the issuance request, trigger transmission of a Transaction Code to the End-User's phone from a service other than Credential issuance, e.g., from a payment service. The End-User would then be asked to enter this Transaction Code into the Wallet and since the Wallet sends this Transaction Code to the Token Endpoint of the Credential Issuer (the attacker), the attacker would get access to the Transaction Code, and access to that other service.

In order to cope with that issue, the Wallet is RECOMMENDED to interact with trusted Credential Issuers only. In that case, the Wallet would not process a Credential Offer with an untrusted issuer URL. The Wallet MAY also show the End-User the endpoint of the Credential Issuer it will be sending the Transaction Code to and ask the End-User for confirmation.

## Credential Lifecycle Management 

The Credential Issuer is supposed to be responsible for the lifecycle of its Credentials. This means the Credential Issuer will invalidate Credentials when it deems appropriate, e.g., if it detects fraudulent behavior.

The Wallet is supposed to detect signs of fraudulent behavior related to the Credential management in the Wallet (e.g., device rooting) and to act upon such signals. Options include Credential revocation at the Credential Issuer and/or invalidation of the key material used to cryptographically bind the Credential to the identifier of the End-User possessing that Credential.

## Proof replay {#key-proof-replay}

If an adversary is able to obtain a key proof, as defined in (#proof-types), the adversary could get a Credential issued that is bound to a key pair controlled by the victim.

Note: For the attacker to be able to present a Credential bound to a replayed key proof to the Verifier, the attacker also needs to obtain the victim's private key. To limit this, servers are RECOMMENDED to check how the Wallet protects the private keys, using mechanisms such as Attestation-Based Client Authentication as defined in [@!I-D.ietf-oauth-attestation-based-client-auth].

The `nonce` parameter is the primary countermeasure against key proof replay. To further narrow down the attack vector, the Credential Issuer SHOULD bind a unique `nonce` parameter to the respective Access Token.

Note: To accommodate for clock offsets, the Credential Issuer server MAY accept proofs that carry an `iat` time in the reasonably near future (on the order of seconds or minutes). Because clock skews between servers and Clients may be large, servers MAY limit key proof lifetimes by using server-provided nonce values containing the time at the server rather than comparing the client-supplied `iat` time to the time at the server. Nonces created in this way yield the same result even in the face of arbitrarily large clock skews.

Server-provided nonces are an effective means for further reducing the chances for successful key proof replay by an attacker. A Wallet can keep using a provided nonce value until the Credential Issuer provides a fresh nonce. This way, the Credential Issuer determines how often a certain nonce can be used. Servers MUST have a clear policy on whether the same key proof can be presented multiple times and for how long, or whether each Credential Request MUST have a fresh key proof.

##  TLS Requirements

Implementations MUST follow [@!BCP195].
Whenever TLS is used, a TLS server certificate check MUST be performed, per [@!RFC6125].

## Protecting the Access Token

Access Tokens represent End-User authorization and consent to issue certain Credential(s). Long-lived Access Tokens giving access to Credentials MUST not be issued unless sender-constrained. Access Tokens with lifetimes longer than 5 minutes are, in general, considered long lived.

To sender-constrain Access Tokens, see the recommendations in Section 4.10.1 in [@!I-D.ietf-oauth-security-topics]. If Bearer Access Tokens are stored by the Wallet, they MUST be stored in a secure manner, for example, encrypted using a key stored in a protected key store.

# Implementation Considerations

## Claims-based Binding of the Credential to the End-User possessing the Credential {#claims-based-binding}

Credentials not cryptographically bound to the identifier of the End-User possessing it (see (#credential-binding)), should be bound to the End-User possessing the Credential, based on the claims included in the Credential.

In claims-based binding, no cryptographic binding material is provided. Instead, the issued Credential includes End-User claims that can be used by the Verifier to verify possession of the Credential by requesting presentation of existing forms of physical or digital identification that includes the same claims (e.g., a driving license or other ID cards in person, or an online ID verification service).

## Binding of the Credential without Cryptographic Binding or Claims-based Binding {#no-binding}

Some Credential Issuers might choose to issue bearer Credentials without either cryptographic binding or claims-based binding because they are meant to be presented without proof of possession.

One such use case is low assurance Credentials, such as coupons or tickets.

Another use case is when the Credential Issuer uses cryptographic schemes that can provide binding to the End-User possessing that Credential without explicit cryptographic material being supplied by the application used by that End-User. For example, in the case of the BBS Signature Scheme, the issued Credential itself is a secret and only a derivation from the Credential is presented to the Verifier. Effectively, the Credential is bound to the Credential Issuer's signature on the Credential, which becomes a shared secret transferred from the Credential Issuer to the End-User.

## Multiple Accesses to the Credential Endpoint

The Credential Endpoint can be accessed multiple times by a Wallet using the same Access Token, even for the same Credential. The Credential Issuer determines if the subsequent successful requests will return the same or an updated Credential, such as having a new expiration time or using the most current End-User claims.

The Credential Issuer MAY also decide to no longer accept the Access Token and a re-authentication or Token Refresh (see [@!RFC6749], Section 6) MAY be required at the Credential Issuer's discretion. The policies between the Credential Endpoint and the Authorization Server that MAY change the behavior of what is returned with a new Access Token are beyond the scope of this specification (see [@!RFC6749, Section 7]).

The action leading to the Wallet performing another Credential Request can also be triggered by a background process, or by the Credential Issuer using an out-of-band mechanism (SMS, email, etc.) to inform the End-User.

## Relationship between the Credential Issuer Identifier in the Metadata and the Issuer Identifier in the Issued Credential

The Credential Issuer Identifier is always a URL using the `https` scheme, as defined in (#credential-issuer-identifier). Depending on the Credential Format, the Issuer identifier in the issued Credential may not be a URL using the `https` scheme. Some other forms that it can take are a DID included in the `issuer` property in a [@VC_DATA] format, or the `Subject` value of the document signer certificate included in the `x5chain` element in an [@ISO.18013-5] format.

When the Issuer identifier in the issued Credential is a DID, a non-exhaustive list of mechanisms the Credential Issuer MAY use to bind to the Credential Issuer Identifier is as follows:

1. Use the [@DIF.Well-Known_DID] Specification to provide binding between a DID and a certain domain.
1. If the Issuer identifier in the issued Credential is an object, add to the object a `credential_issuer` claim, as defined in (#credential-issuer-identifier).

The Wallet MAY check the binding between the Credential Issuer Identifier and the Issuer identifier in the issued Credential.

## Refreshing Issued Credentials

After a Verifiable Credential has been issued to the Holder, claim values about the subject of a Credential or a signature on the Credential may need to be updated. There are two possible mechanisms to do so.

First, the Wallet may receive an updated version of a Credential from a Credential Endpoint using a valid Access Token. This does not involve interaction with the End-User. If the Credential Issuer issued a Refresh Token to the Wallet, the Wallet would obtain a fresh Access Token by making a request to the Token Endpoint, as defined in Section 6 of [@!RFC6749].

Second, the Credential Issuer can reissue the Credential by starting the issuance process from the beginning. This would involve interaction with the End-User. A Credential needs to be reissued if the Wallet does not have a valid Access Token or a valid Refresh Token. With this approach, when a new Credential is issued, the Wallet might need to check if it already has a Credential of the same type and, if necessary, delete the old Credential. Otherwise, the Wallet might end up with more than one Credential of the same type, without knowing which one is the latest.

Credential Refresh can be initiated by the Wallet independently from the Credential Issuer, or the Credential Issuer can send a signal to the Wallet asking it to request Credential refresh. How the Credential Issuer sends such a signal is out of scope of this specification.

It is up to the Credential Issuer whether to update both the signature and the claim values, or only the signature.

# Privacy Considerations

When [@!RFC9396] is used, the Privacy Considerations of that specification also apply.

The privacy principles of [@ISO.29100] should be adhered to.

## User Consent

The Credential Issuer SHOULD obtain the End-User's consent before issuing Credential(s)
to the Wallet. It SHOULD be made clear to the End-User what information is being included in the
Credential(s) and for what purpose.

## Minimum Disclosure

To ensure minimum disclosure and prevent Verifiers from obtaining claims unnecessary for the
transaction at hand, when issuing Credentials that are intended to be created once and then used a
number of times by the End-User, the Credential Issuers and the Wallets SHOULD implement
Credential Formats that support selective disclosure, or consider issuing a separate Credential
for each user claim.

## Storage of the Credentials

To prevent a leak of End-User data, especially when it is signed, which risks revealing private data of End-Users to
third parties, systems implementing this specification SHOULD be designed to minimize the amount
of End-User data that is stored. All involved parties SHOULD store Verifiable Credentials
containing privacy-sensitive data only for as long as needed, including in log files. Any logging of
End-User data should be carefully considered as to whether it is necessary at all. The time logs are retained
for should be minimized.

After Issuance, Credential Issuers SHOULD NOT store the Issuer-signed Credentials if they
contain privacy-sensitive data. Wallets SHOULD store Credentials only in encrypted form, and,
wherever possible, use hardware-backed encryption. Wallets SHOULD not store
Credentials longer than needed.

## Correlation 

### Unique Values Encoded in the Credential

Issuance/presentation or two presentation sessions by the same End-User can be linked on the basis of
unique values encoded in the Credential (End-User claims, identifiers, Issuer signature, etc.) either by colluding Issuer/Verifier or Verifier/Verifier pairs, or by the same Verifier.

To prevent these types of correlation, Credential Issuers and Wallets SHOULD use
methods, including but not limited to the following ones:

* Issue a batch of Credentials with the same Credential Dataset to facilitate the use of a unique Credential per presentation or per Verifier. This approach solely aids in achieving Verifier-to-Verifier unlinkability.
* Use cryptographic schemes that can provide non-correlation.

Credential Issuers specifically SHOULD discard values that can be used in collusion with a Verifier to track a user, such as the Issuer's signature or cryptographic key material to which an issued credential was bound to.

### Credential Offer

The Privacy Considerations in Section 11.2 of [@!RFC9101] apply to the `credential_offer` and
`credential_offer_uri` parameters defined in (#credential-offer).

### Authorization Request

The Wallet SHOULD NOT include potentially sensitive information in the Authorization Request,
for example, by including clear-text session information as a `state` parameter value or encoding
it in a `redirect_uri` parameter. A third party may observe such information through browser
history, etc. and correlate the user's activity using it.

## Identifying the Credential Issuer

Information in the credential identifying a particular Credential Issuer, such as a Credential Issuer Identifier,
issuer's certificate, or issuer's public key may reveal information about the End-User.

For example, when a military organization or a drug rehabilitation center issues a vaccine
credential, verifiers can deduce that the owner of the Wallet storing such Credential is a
military member or may have a substance use disorder.

In addition, when a Credential Issuer issues only one type of Credential, it might have privacy implications,
because if the Wallet has a Credential issued by that Issuer, its type and claim names can be
determined.

For example, if the National Cancer Institute only issued Credentials with cancer registry
information, it is possible to deduce that the owner of the Wallet storing such Credential is a
cancer patient.

To mitigate these issues, a group of organizations may elect to use a common Credential Issuer,
such that any credentials issued by this Issuer cannot be attributed to a particular organization
through identifiers of the Credential Issuers alone. A group signature scheme may also be used
instead of an individual signature.

When a common Credential Issuer is used, appropriate guardrails need to be in place to prevent
one organization from issuing illegitimate credentials on behalf of other organizations.

## Identifying the Wallet

There is a potential for leaking information about the Wallet to third parties when the
Wallet reacts to a Credential Offer. An attacker may send Credential Offers using different
custom URL schemes or claimed https urls, see if the
Wallet reacts (e.g., whether the wallet retrieves Credential Issuer metadata hosted by an
attacker's server), and, therefore, learn which Wallet is installed. To avoid this, the Wallet SHOULD
require user interaction or establish trust in the Issuer before fetching any `credential_offer_uri`
or acting on the received Credential Offer.

## Untrusted Wallets

The Wallet transmits and stores sensitive information about the End-User. To ensure that the
Wallet can handle those appropriately (i.e., according to a certain trust framework or a
regulation), the Credential Issuer should properly authenticate the Wallet and ensure it is a trusted entity. For more details, see (#trust-between-wallet-and-issuer).

{backmatter}

<reference anchor="DID-Core" target="https://www.w3.org/TR/did-core/">
        <front>
        <title>Decentralized Identifiers (DIDs) v1.0</title>
        <author fullname="Manu Sporny">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="Amy Guy">
            <organization>Digital Bazaar</organization>
        </author>
        <author fullname="Markus Sabadello">
            <organization>Danube Tech</organization>
        </author>
        <author fullname="Drummond Reed">
            <organization>Evernym/Avast</organization>
        </author>
        <date day="19" month="Jul" year="2022"/>
        </front>
</reference>

<reference anchor="VC_DATA" target="https://www.w3.org/TR/vc-data-model">
  <front>
    <title>Verifiable Credentials Data Model 1.1</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Grant Noble">
      <organization>ConsenSys</organization>
    </author>
    <author fullname="Dave Longley">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Daniel C. Burnett">
      <organization>ConsenSys</organization>
    </author>
    <author fullname="Brent Zundel">
      <organization>Evernym</organization>
    </author>
    <author fullname="Kyle Den Hartog">
      <organization>Mattr</organization>
    </author>
   <date day="3" month="March" year="2022"/>
  </front>
</reference>

<reference anchor="VC_DATA_2.0" target="https://www.w3.org/TR/vc-data-model-2.0">
  <front>
    <title>Verifiable Credentials Data Model 2.0</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Ted Thibodeau Jr">
      <organization>OpenLink Software</organization>
    </author>
    <author fullname="Ivan Herman">
      <organization>W3C</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Invited Expert</organization>
    </author>
    <author fullname="Gabe Cohen">
      <organization>Block</organization>
    </author>
   <date day="27" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="VC_Data_Integrity" target="https://w3c.github.io/vc-data-integrity/">
  <front>
    <title>Verifiable Credential Data Integrity 1.0</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Dave Longley">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Greg Bernstein">
      <organization>Invited Expert</organization>
    </author>
    <author fullname="Dmitri Zagidulin">
      <organization>Invited Expert</organization>
    </author>
    <author fullname="Sebastian Crane">
      <organization>Invited Expert</organization>
    </author>
   <date day="14" month="November" year="2023"/>
  </front>
</reference>

<reference anchor="USASCII">
        <front>
          <title>Coded Character Set -- 7-bit American Standard Code for Information Interchange</title>
          <author>
            <organization>American National Standards Institute</organization>
          </author>
          <date year="1986"/>
        </front>
</reference>

<reference anchor="SIOPv2" target="https://openid.net/specs/openid-connect-self-issued-v2-1_0.html">
  <front>
    <title>Self-Issued OpenID Provider V2</title>
    <author fullname="Kristina Yasuda">
      <organization>German Federal Agency for Disruptive Innovation (SPRIND)</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Self-Issued Consulting</organization>
    </author>
    <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
      <organization>German Federal Agency for Disruptive Innovation (SPRIND)</organization>
    </author>
   <date day="28" month="November" year="2023"/>
  </front>
</reference>

<reference anchor="BCP195" target="https://www.rfc-editor.org/info/bcp195">
        <front>
          <title>BCP195</title>
          <author>
            <organization>IETF</organization>
          </author>
          <date month="November" year="2022"/>
        </front>
</reference>

<reference anchor="OpenID.Core" target="http://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 2</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="M." surname="Jones" fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="C." surname="Mortimore" fullname="Chuck Mortimore">
      <organization>Salesforce</organization>
    </author>
   <date day="15" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="DIF.Well-Known_DID" target="https://identity.foundation/specs/did-configuration/">
        <front>
          <title>Well Known DID Configuration</title>
      <author fullname="Daniel Buchner">
            <organization>Microsoft</organization>
          </author>
          <author fullname="Orie Steele">
            <organization>Transmute</organization>
          </author>
          <author fullname="Tobias Looker">
            <organization>Mattr</organization>
          </author>
        </front>
</reference>

<reference anchor="CSS-Color" target="https://www.w3.org/TR/css-color-3">
      <front>
        <title>CSS Color Module Level 3</title>
        <author initials="T." surname="Çelik" fullname="Tantek Çelik">
         <organization>Mozilla Corporation</organization>
        </author>
        <author initials="C." surname="Lilley" fullname="Chris Lilley">
          <organization>W3C</organization>
        </author>
        <author initials="D." surname="Baron" fullname="L. David Baron">
          <organization>W3C Invited Expert</organization>
       </author>
       <date day="18" month="January" year="2022"/>
      </front>
</reference>

<reference anchor="JSON-LD" target="https://www.w3.org/TR/json-ld11/">
      <front>
        <title>JSON-LD 1.1: A JSON-based Serialization for Linked Data.</title>
        <author fullname="Gregg Kellogg">
        </author>
        <author fullname="Manu Sporny">
        </author>
        <author fullname="Dave Longley">
       </author>
       <author fullname="Markus Lanthaler">
       </author>
       <author fullname="Pierre-Antoine Champin">
       </author>
       <author fullname="Niklas Lindström">
       </author>
       <date day="16" month="July" year="2020"/>
      </front>
</reference>

<reference anchor="ISO.18013-5" target="https://www.iso.org/standard/69084.html">
        <front>
          <title>ISO/IEC 18013-5:2021 Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL)  application</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date Month="September" year="2021"/>
        </front>
</reference>

<reference anchor="IANA.JOSE.ALGS" target="https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms">
        <front>
          <title>JSON Web Signature and Encryption Algorithms</title>
          <author>
            <organization>IANA</organization>
          </author>
        </front>
</reference>

<reference anchor="IANA.COSE.ALGS" target="https://www.iana.org/assignments/cose/cose.xhtml#algorithms">
        <front>
          <title>COSE Algorithms</title>
          <author>
            <organization>IANA</organization>
          </author>
        </front>
</reference>

<reference anchor="OpenID4VP" target="https://openid.net/specs/openid-4-verifiable-presentations-1_0.html">
      <front>
        <title>OpenID for Verifiable Presentations</title>
        <author initials="O." surname="Terbu" fullname="Oliver Terbu">
         <organization>Mattr</organization>
        </author>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
          <organization>German Federal Agency for Disruptive Innovation (SPRIND)</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>German Federal Agency for Disruptive Innovation (SPRIND)</organization>
        </author>
        <author initials="T." surname="Looker" fullname="Tobias Looker">
          <organization>Mattr</organization>
        </author>
       <date day="29" month="November" year="2022"/>
      </front>
</reference>

<reference anchor="DID_Specification_Registries" target="https://www.w3.org/TR/did-spec-registries/">
        <front>
          <title>DID Specification Registries</title>
      <author fullname="Orie Steele">
            <organization>Transmute</organization>
          </author>
          <author fullname="Manu Sporny">
            <organization>Digital Bazaar</organization>
          </author>
         <date day="11" month="September" year="2023"/>
        </front>
</reference>

<reference anchor="LD_Suite_Registry" target="https://w3c-ccg.github.io/ld-cryptosuite-registry/">
        <front>
          <title>Linked Data Cryptographic Suite Registry</title>
          <author fullname="Manu Sporny">
            <organization>Digital Bazaar</organization>
          </author>
          <author fullname="Drummond Reed">
            <organization>Evernym</organization>
          </author>
          <author fullname="Orie Steele">
            <organization>Transmute</organization>
          </author>
         <date day="29" month="December" year="2020"/>
        </front>
</reference>

<reference anchor="OpenID.Federation" target="https://openid.net/specs/openid-federation-1_0.html">
        <front>
          <title>OpenID Federation 1.0</title>
		  <author fullname="R. Hedberg, Ed.">
            <organization>independent</organization>
          </author>
          <author fullname="Michael B. Jones">
            <organization>Self-Issued Consulting</organization>
          </author>
          <author fullname="A. Solberg">
            <organization>Sikt</organization>
          </author>
          <author fullname="John Bradley">
            <organization>Yubico</organization>
          </author>
          <author fullname="Giuseppe De Marco">
            <organization>independent</organization>
          </author>
          <author fullname="Vladimir Dzhuvinov">
            <organization>Connect2id</organization>
          </author>
          <date day="4" month="December" year="2023"/>
        </front>
</reference>

<reference anchor="IANA.OAuth.Parameters" target="https://www.iana.org/assignments/oauth-parameters">
  <front>
    <title>OAuth Parameters</title>
    <author>
      <organization>IANA</organization>
    </author>
    <date/>
  </front>
</reference>

<reference anchor="IANA.MediaTypes" target="https://www.iana.org/assignments/media-types">
  <front>
    <title>Media Types</title>
    <author>
      <organization>IANA</organization>
    </author>
    <date/>
  </front>
</reference>

<reference anchor="eIDAS" target="https://eur-lex.europa.eu/legal-content/EN/TXT/HTML/?uri=CELEX:32014R0910">
  <front>
    <title>REGULATION (EU) No 910/2014 OF THE EUROPEAN PARLIAMENT AND OF THE COUNCIL on electronic identification and trust services for electronic transactions in the internal market and repealing Directive 1999/93/EC</title>
    <author surname="European Parliament">
      <organization>European Parliament</organization>
    </author>
    <date year="2014" month="July" day="23"></date>
  </front>
</reference>

<reference anchor="ISO.29100" target="https://standards.iso.org/ittf/PubliclyAvailableStandards/index.html">
  <front>
    <author fullname="ISO"></author>
    <title>ISO/IEC 29100:2011 Information technology — Security techniques — Privacy framework</title>
  </front>
</reference>

# Credential Format Profiles {#format-profiles}

This specification defines several extension points to accommodate the differences across Credential Formats. Sets of Credential Format-specific parameters or claims referred to as Credential Format Profiles are identified by the Credential Format Identifier and used at these extension points.

This section defines Credential Format Profiles for a few of the commonly used Credential Formats. Other specifications or deployments can define their own Credential Format Profiles. It is RECOMMENDED that new Credential Format Profiles use the media type of the particular Credential Format for the Credential Format Identifier.

## W3C Verifiable Credentials

Sections 6.1 and 6.2 of [@VC_DATA] define how Verifiable Credentials MAY or MAY NOT use JSON-LD [@JSON-LD]. As acknowledged in Section 4.1 of [@VC_DATA], implementations can behave differently regarding processing of the `@context` property whether JSON-LD is used or not.

This specification therefore differentiates between the following three Credential Formats for W3C Verifiable Credentials:

* VC signed as a JWT, not using JSON-LD (`jwt_vc_json`)
* VC signed as a JWT, using JSON-LD (`jwt_vc_json-ld`)
* VC secured using Data Integrity, using JSON-LD, with a proof suite requiring Linked Data canonicalization (`ldp_vc`)

Note: VCs secured using Data Integrity MAY NOT necessarily use JSON-LD and MAY NOT necessarily use proof suites requiring Linked Data canonicalization. Credential Format Profiles for them may be defined in the future versions of this specification.

Distinct Credential Format Identifiers, extension parameters/claims, and processing rules are defined for each of the above-mentioned Credential Formats.

It is on purpose that the Credential Offer does not contain `credentialSubject` property, while Authorization Details and Credential Request do. This is because this property is meant to be used by the Wallet to specify which claims it is requesting to be issued out of all the claims the Credential Issuer is capable of issuing for this particular Credential (data minimization), while Credential Offer is a mere "invitation" from the Credential Issuer to the Wallet to start the issuance flow.

### VC Signed as a JWT, Not Using JSON-LD {#jwt-vc-json}

#### Format Identifier

The Credential Format Identifier is `jwt_vc_json`.

When the `format` value is `jwt_vc_json`, the entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules.

#### Credential Issuer Metadata {#server-metadata-jwt-vc-json}

Cryptographic algorithm names used in the `credential_signing_alg_values_supported` parameter SHOULD be one of those defined in [@IANA.JOSE.ALGS].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters).

* `credential_definition`: REQUIRED. Object containing the detailed description of the Credential type. It consists of the following parameters defined by this specification:
  * `type`: REQUIRED. Array designating the types a certain Credential type supports, according to [@VC_DATA], Section 4.3.
  * `credentialSubject`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. To express the specifics about the claim, the most deeply nested value MAY be an object that includes the following parameters defined by this specification (other parameters MAY also be used):
      * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the Credential Issuer will always include this claim in the issued Credential. If set to `false`, the claim is not included in the issued Credential if the wallet did not request the inclusion of the claim, and/or if the Credential Issuer chose to not include the claim. If the `mandatory` parameter is omitted, the default value is `false`.
      * `value_type`: OPTIONAL. String value determining the type of value of the claim. Valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image). Other values MAY also be used.
      * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
          * `name`: OPTIONAL. String value of a display name for the claim.
          * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. Array of the claim name values that lists them in the order they should be displayed by the Wallet.

The following is a non-normative example of an object containing the `credential_configurations_supported` parameter for Credential Format `jwt_vc_json`:

<{{examples/credential_metadata_jwt_vc_json.json}}

#### Authorization Details {#authorization-jwt-vc-json}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential Format.

* `credential_definition`: OPTIONAL. Object containing a detailed description of the Credential consisting of the following parameter:
  * `type`: OPTIONAL. Array as defined in (#server-metadata-jwt-vc-json). This claim contains the type values the Wallet requests authorization for at the Credential Issuer. It MUST be present if the claim `format` is present in the root of the authorization details object. It MUST not be present otherwise. 
  * `credentialSubject`: OPTIONAL. Object as defined in (#server-metadata-sd-jwt-vc) excluding the `display` and `value_type` parameters. `mandatory` parameter here is used by the Wallet to indicate to the Issuer that it only accepts Credential(s) issued with those claim(s).

The following is a non-normative example of an authorization details object with Credential Format `jwt_vc_json`:

<{{examples/authorization_details_jwt_vc_json.json}}

#### Credential Request

The following additional parameters are defined for Credential Requests and this Credential Format.  

* `credential_definition`: REQUIRED when the `format` parameter is present in the Credential Request. It MUST NOT be used otherwise. It is an object containing the detailed description of the Credential type. It consists of the following parameters defined by this specification:
  * `type`: REQUIRED. Array as defined in (#server-metadata-jwt-vc-json). The credential issued by the Credential Issuer MUST contain at least the values listed in this claim.
  * `credentialSubject`: OPTIONAL. Object as defined in (#authorization-jwt-vc-json).

The following is a non-normative example of a Credential Request with Credential Format `jwt_vc_json`:

<{{examples/credential_request_jwt_vc_json_with_claims.json}}

#### Credential Response {#credential-response-jwt-vc-json}

The value of the `credential` claim in the Credential Response MUST be a JWT. Credentials of this format are already a sequence of base64url-encoded values separated by period characters and MUST NOT be re-encoded. 

The following is a non-normative example of a Credential Response with Credential Format `jwt_vc_json`:

<{{examples/credential_response_jwt_vc_json.txt}}

### VC Secured using Data Integrity, using JSON-LD, with a Proof Suite Requiring Linked Data Canonicalization

#### Format Identifier

The Credential Format Identifier is `ldp_vc`.

When the `format` value is `ldp_vc`, the entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules. 

The `@context` value in the `credential_definition` object can be used by the Wallet to check whether it supports a certain VC or not. If necessary, the Wallet could apply JSON-LD processing to the Credential issued by the Credential Issuer.

Note: Data Integrity used to be called Linked Data Proofs, hence the "ldp" in the Credential Format Identifier.

#### Credential Issuer Metadata {#server-metadata-ldp-vc}

Cryptographic algorithm names used in the `credential_signing_alg_values_supported` parameter SHOULD be one of those defined in [@LD_Suite_Registry].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters):

* `credential_definition`: REQUIRED. Object containing the detailed description of the Credential type. It consists of the following parameters defined by this specification:
  * `@context`: REQUIRED. Array as defined in [@VC_DATA], Section 4.1.
  * `type`: REQUIRED. Array designating the types a certain credential type supports, according to [@VC_DATA], Section 4.3.
  * `credentialSubject`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. To express the specifics about the claim, the most deeply nested value MAY be an object that includes the following parameters defined by this specification (other parameters MAY also be included):
      * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the Credential Issuer will always include this claim in the issued Credential. If set to `false`, the claim is not included in the issued Credential if the wallet did not request the inclusion of the claim, and/or if the Credential Issuer chose to not include the claim. If the `mandatory` parameter is omitted, the default value is `false`.
      * `value_type`: OPTIONAL. String value determining the type of value of the claim. Valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image). Other values MAY also be used.
      * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
          * `name`: OPTIONAL. String value of a display name for the claim.
          * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. Array of the claim name values that lists them in the order they should be displayed by the Wallet.


The following is a non-normative example of an object containing the `credential_configurations_supported` parameter for Credential Format `ldp_vc`:

<{{examples/credential_metadata_ldp_vc.json}}

#### Authorization Details {#authorization-ldp-vc}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential Format.  

* `credential_definition`: OPTIONAL. Object containing the detailed description of the Credential. It consists of the following parameters defined by this specification:
    * `@context`: OPTIONAL. Array as defined in (#server-metadata-ldp-vc). It MUST only be present if the `format` claim is present in the root of the authorization details object. It MUST not be present otherwise. 
    * `type`: OPTIONAL. Array as defined in (#server-metadata-ldp-vc).  This claim contains the type values the Wallet requests authorization for at the Credential Issuer. MUST only be present if the `@context` claim is present. 
    * `credentialSubject`: OPTIONAL. Object as defined in (#server-metadata-sd-jwt-vc) excluding the `display` and `value_type` parameters. `mandatory` parameter here is used by the Wallet to indicate to the Issuer that it only accepts Credential(s) issued with those claim(s).

The following is a non-normative example of an authorization details object with Credential Format `ldp_vc`:

<{{examples/authorization_details_ldp_vc.json}}

#### Credential Request {#credential-request-ldp-vc}

The following additional parameters are defined for Credential Requests and this Credential Format.  

* `credential_definition`: REQUIRED when the `format` parameter is present in the Credential Request. It MUST NOT be used otherwise. It is an object containing the detailed description of the Credential type. It consists of the following parameters defined by this specification:
  * `@context`: REQUIRED. Array as defined in (#server-metadata-ldp-vc).
  * `type`: REQUIRED. Array as defined in (#server-metadata-ldp-vc). The Credential issued by the Credential Issuer MUST contain at least the values listed in this claim.
  * `credentialSubject`: OPTIONAL. Object as defined in (#authorization-ldp-vc).

The following is a non-normative example of a Credential Request with Credential Format `ldp_vc` with the key proof type `jwt`:

<{{examples/credential_request_ldp_vc.json}}

The following is a non-normative example of a Credential Request with Credential Format `ldp_vc` with the key proof type `ldp_vp`:

<{{examples/credential_request_ldp_vc_vp.json}}

#### Credential Response

The value of the `credential` claim in the Credential Response MUST be a JSON object. Credentials of this format MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response with Credential Format `ldp_vc`:

<{{examples/credential_response_ldp_vc.txt}}

### VC signed as a JWT, Using JSON-LD

#### Format Identifier

The Credential Format Identifier is `jwt_vc_json-ld`.

When the `format` value is `jwt_vc_json-ld`, the entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules.

The `@context` value in the `credential_definition` parameter can be used by the Wallet to check whether it supports a certain VC or not. If necessary, the Wallet could apply JSON-LD processing to the Credential issued by the Credential Issuer.

#### Credential Issuer Metadata

The definitions in (#server-metadata-ldp-vc) apply for metadata of Credentials of this type as well. 

#### Authorization Details

The definitions in (#authorization-ldp-vc) apply for Credentials of this type as well.

#### Credential Request

The definitions in (#credential-request-ldp-vc) apply for Credentials of this type as well.

#### Credential Response

The definitions in (#credential-response-jwt-vc-json) apply for Credentials of this type as well.

## ISO mDL

This section defines a Credential Format Profile for Credentials complying with [@!ISO.18013-5].

### Format Identifier

The Credential Format Identifier is `mso_mdoc`. This refers to the Mobile Security Object (MSO) which secures the mdoc data model encoded as CBOR.

### Credential Issuer Metadata {#server-metadata-mso-mdoc}

Cryptographic algorithm names used in the `credential_signing_alg_values_supported` parameter SHOULD be one of those defined in [@!ISO.18013-5].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters).

* `doctype`: REQUIRED. String identifying the Credential type, as defined in [@!ISO.18013-5].
* `claims`: OPTIONAL. Object containing a list of name/value pairs, where the name is a certain `namespace` as defined in [@!ISO.18013-5] (or any profile of it), and the value is an object. This object also contains a list of name/value pairs, where the name is a claim name value that is defined in the respective namespace and is offered in the Credential. The value is an object detailing the specifics of the claim with the following non-exhaustive list of parameters that MAY be included:
    * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the Credential Issuer will always include this claim in the issued Credential. If set to `false`, the claim is not included in the issued Credential if the wallet did not request the inclusion of the claim, and/or if the Credential Issuer chose to not include the claim. If the `mandatory` parameter is omitted, the default value is `false`.
    * `value_type`: OPTIONAL. String value determining the type of value of the claim. Valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image). Other values MAY also be used.
    * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. Array of namespaced claim name values that lists them in the order they should be displayed by the Wallet. The values MUST be two strings separated by a tilde ('~') character, where the first string is a namespace value and a second is a claim name value. For example, `org.iso.18013.5.1~given_name".

The following is a non-normative example of an object containing the `credential_configurations_supported` parameter for Credential Format `mso_mdoc`:

<{{examples/credential_metadata_mso_mdoc.json}}

### Authorization Details

The following additional claims are defined for authorization details of type `openid_credential` and this Credential Format.

* `doctype`: OPTIONAL. String as defined in (#server-metadata-mso-mdoc). This claim contains the type value the Wallet requests authorization for at the Credential Issuer. It MUST only be present if the `format` claim is present. It MUST not be present otherwise. 
* `claims`: OPTIONAL. Object as defined in (#server-metadata-sd-jwt-vc) excluding the `display` and `value_type` parameters. `mandatory` parameter here is used by the Wallet to indicate to the Issuer that it only accepts Credential(s) issued with those claim(s).

The following is a non-normative example of an authorization details object with Credential Format `mso_mdoc`:

<{{examples/authorization_details_mso_doc.json}}

### Credential Request

The following additional parameters are defined for Credential Requests and this Credential Format.  

* `doctype`: REQUIRED when the `format` parameter is present in the Credential Request. It MUST NOT be used otherwise. It is a string as defined in (#server-metadata-mso-mdoc). The Credential issued by the Credential Issuer MUST contain at least the values listed in this claim.
* `claims`: OPTIONAL. Object as defined in (#server-metadata-mso-mdoc).

The following is a non-normative example of a Credential Request with Credential Format `mso_mdoc`:

<{{examples/credential_request_iso_mdl_with_claims.json}}

### Credential Response

The value of the `credential` claim in the Credential Response MUST be a string that is the base64url-encoded representation of the CBOR-encoded `IssuerSigned` structure, as defined in [@!ISO.18013-5]. This structure SHOULD contain all `Namespaces` and `IssuerSignedItems` that are included in the `AuthorizedNamespaces` of the `MobileSecurityObject`.

The following is a non-normative example of a Credential Response containing a Credential of format `mso_mdoc`.

<{{examples/credential_response_mso_mdoc.txt}}

## IETF SD-JWT VC

This section defines a Credential Format Profile for Credentials complying with [@!I-D.ietf-oauth-sd-jwt-vc].

### Format Identifier

The Credential Format Identifier is `vc+sd-jwt`.

### Credential Issuer Metadata {#server-metadata-sd-jwt-vc}

Cryptographic algorithm names used in the `credential_signing_alg_values_supported` parameter SHOULD be one of those defined in [@IANA.JOSE.ALGS].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters).


* `vct`: REQUIRED. String designating the type of a Credential, as defined in [@!I-D.ietf-oauth-sd-jwt-vc].
* `claims`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim about the subject offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. To express the specifics about the claim, the most deeply nested value MAY be an object that includes the following parameters defined by this specification:
    * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued Credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
    * `value_type`: OPTIONAL. String value determining the type of value of the claim. Valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image). Other values MAY also be used.
    * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. An array of the claim name values that lists them in the order they should be displayed by the Wallet.

The following is a non-normative example of an object comprising the `credential_configurations_supported` parameter for Credential Format `vc+sd-jwt`.

<{{examples/credential_metadata_sd_jwt_vc.json}}

### Authorization Details {#authorization-sd-jwt-vc}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential Format.

* `vct`: REQUIRED. String as defined in (#server-metadata-sd-jwt-vc). This claim contains the type values the Wallet requests authorization for at the Credential Issuer. It MUST only be present if the `format` claim is present. It MUST not be present otherwise.
* `claims`: OPTIONAL. Object as defined in (#server-metadata-sd-jwt-vc) excluding the `display` and `value_type` parameters. `mandatory` parameter here is used by the Wallet to indicate to the Issuer that it only accepts Credential(s) issued with those claim(s).

The following is a non-normative example of an authorization details object with Credential Format `vc+sd-jwt`.

<{{examples/authorization_details_sd_jwt_vc.json}}

### Credential Request

The following additional parameters are defined for Credential Requests and this Credential Format.

* `vct`: REQUIRED when the `format` parameter is present in the Credential Request. It MUST NOT be used otherwise. It is a string as defined in (#server-metadata-sd-jwt-vc). This claim contains the type value of the Credential that the Wallet requests the Credential Issuer to issue.
* `claims`: OPTIONAL. An object as defined in (#server-metadata-sd-jwt-vc).

The following is a non-normative example of a Credential Request with Credential Format `vc+sd-jwt`.

<{{examples/credential_request_sd_jwt_vc.json}}

### Credential Response {#credential-response-jwt-vc-json}

The value of the `credential` claim in the Credential Response MUST be a string that is an SD-JWT VC. Credentials of this format are already suitable for transfer and, therefore, they need not and MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response containing a Credential of format `vc+sd-jwt`.

<{{examples/credential_response_sd_jwt_vc.txt}}

# IANA Considerations

## Sub-Namespace Registration

This specification registers the following URN in the IANA "OAuth URI" registry [@!IANA.OAuth.Parameters] established by [@!RFC6755].

* URN: urn:ietf:params:oauth:grant-type:pre-authorized_code
* Common Name: Pre-Authorized Code
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-request) of this specification

## OAuth Parameters Registry

This specification registers the following parameter names in the IANA "OAuth Parameters" registry [@!IANA.OAuth.Parameters] established by [@!RFC6749].

* Parameter Name: wallet_issuer
* Parameter Usage Location: authorization request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential-authz-request) of this specification

* Parameter Name: user_hint
* Parameter Usage Location: authorization request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential-authz-request) of this specification

* Parameter Name: issuer_state
* Parameter Usage Location: authorization request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential-authz-request) of this specification

* Parameter Name: pre-authorized_code
* Parameter Usage Location: token request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-request) of this specification

* Parameter Name: tx_code
* Parameter Usage Location: token request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-request) of this specification

## OAuth Dynamic Client Registration Metadata Registry

This specification registers the following client metadata name in the IANA "OAuth Dynamic Client Registration Metadata" registry [@!IANA.OAuth.Parameters] established by [@!RFC7591].

* Client Metadata Name: credential_offer_endpoint
* Client Metadata Description: Credential Offer Endpoint
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential-offer-endpoint) of this specification


## Well-Known URI Registry

This specification registers the following well-known URI in the IANA "Well-Known URI" registry established by [@!RFC5785].

* URI suffix: openid-credential-issuer
* Change controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Specification document: (#credential-issuer-wellknown) of this document
* Related information: (none)

## Media Types Registry

This specification registers the following media types in the IANA "Media Types" registry [@!IANA.MediaTypes] in the manner described in [@!RFC6838].

* Type name: `application`
* Subtype name: `openid4vci-proof+jwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Uses JWS Compact Serialization, as specified in [@!RFC7515].
* Security considerations: See the Security Considerations in [@!RFC7519].
* Interoperability considerations: n/a
* Published specification: (#jwt-proof-type) of this specification
* Applications that use this media type: Applications that issue and store verifiable credentials
* Additional information:
  - Magic number(s): n/a
  - File extension(s): n/a
  - Macintosh file type code(s): n/a
* Person & email address to contact for further information: Torsten Lodderstedt, torsten@lodderstedt.net
* Intended usage: COMMON
* Restrictions on usage: none
* Author: Torsten Lodderstedt, torsten@lodderstedt.net
* Change controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Provisional registration? No


# Use Cases

This is a non-exhaustive list of sample use cases.

## Credential Offer - Same-Device {#use-case-1}

While browsing the university's home page, the End-User finds a link "request your digital diploma". The End-User clicks on this link and is redirected to a digital Wallet. The Wallet notifies the End-User that a Credential Issuer offered to issue a diploma Credential. The End-User confirms this inquiry and is taken to the university's Credential issuance service's End-User experience. Upon successful authentication at the university and consent to the issuance of a digital diploma, the End-User is redirected back to the Wallet. Here, the End-User can verify the successful creation of the digital diploma.

## Credential Offer - Cross-Device (with Information Pre-Submitted by the End-User) {#use-case-2}

The End-User is starting a job at a new employer. The employer requests the End-User to upload specific documents to the employee portal. After a few days, the End-User receives an email from the employer indicating that the employee Credential is ready to be claimed and provides instructions to scan a presented QR code for its retrieval. The End-User scans the QR code with a smartphone, which opens the Wallet. Meanwhile, the End-User has received a text message with a Transaction Code to the smartphone. After entering the Transaction Code in the Wallet for security reasons, the End-User approves the Credential issuance, and receives the Credential in the Wallet.

## Credential Offer - Cross-Device & Deferred {#use-case-3}

The End-User intends to acquire a digital criminal record. This involves a visit to the local administration's office to request the official criminal record be issued as a digital Credential. After presenting an ID document, the End-User is prompted to scan a QR code using the Wallet and is informed that the issuance of the Credential will require some time, due to necessary background checks by the authority.

While using the Wallet, the End-User notices an indication that the issuance of the digital record is in progress. After a few days, the End-User receives a notification from the Wallet indicating that the requested Credential was successfully issued. Upon opening the Wallet, the End-User is queried about the download of the Credential. After confirmation, the Wallet fetches and saves the new Credential.

## Wallet-Initiated Issuance during Presentation {#use-case-4}

An End-User comes across a Verifier app that is requesting the End-User to present a Credential, e.g., a driving license. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet selects a Credential Issuer capable of issuing the missing Credential and, upon End-User consent, sends the End-User to the Credential Issuer's End-User experience (Web site or app). Once authenticated and consent is provided for the issuance of the Credential into the Wallet, the End-User is redirected back to the Wallet. The Wallet informs the End-User that Credential was successfully issued into the Wallet and is ready to be presented to the Verifier app that originally requested presentation of that Credential.

## Wallet-Initiated Issuance during Presentation (Requires Presentation of Additional Credentials During Issuance) {#use-case-5}

An End-User comes across a Verifier app that is requesting the End-User to present a Credential, e.g., a university diploma. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet then offers the End-User a list of Credential Issuers, which might be based on a Credential Issuer list curated by the Wallet provider. The End-User selects the university of graduation and is subsequently redirected to the corresponding university's website or app.

The End-User logs into the university, which identifies that the corresponding End-User account is not yet verified. Among various identification options, the End-User opts to present a Credential from the Wallet. The End-User is redirected back to the Wallet to consent to present the requested Credential(s) to the university. Following this, the End-User is redirected back to the university End-User experience. Based on the presented Credential, the university finalizes the End-User verification, retrieves data about the End-User from its database, and proposes to issue a diploma as a Verifiable Credential.

Upon providing consent, the End-User is sent back to the Wallet. The Wallet informs the End-User that the Credential was successfully issued into the Wallet and is ready to be presented to the Verifier app that originally requested presentation of that Credential.

## Wallet-Initiated Issuance after Installation {#use-case-6}

The End-User installs a new Wallet and opens it. The Wallet offers the End-User a selection of Credentials that the End-User may obtain from a Credential Issuer, e.g. a national identity Credential, a mobile driving license, or a public transport ticket. The corresponding Credential Issuers (and their URLs) are pre-configured by the Wallet or follow some discovery processes that are out of scope for this specification. By clicking on one of these options corresponding to the Credentials available for issuance, the issuance process starts using a flow supported by the Credential Issuer (Pre-Authorized Code flow or Authorization Code flow).

Wallet Providers may also provide a market place where Issuers can register to be found for Wallet-initiated flows.

# Acknowledgements {#Acknowledgements}

We would like to thank Richard Barnes, Paul Bastian, Vittorio Bertocci, Christian Bormann, John Bradley, Brian Campbell, Gabe Cohen, David Chadwick, Andrii Deinega, Giuseppe De Marco, Mark Dobrinic, Daniel Fett, Pedro Felix, George Fletcher, Christian Fries, Timo Glasta, Mark Haine, Fabian Hauck, Roland Hedberg, Joseph Heenan, Alen Horvat, Andrew Hughes, Jacob Ideskog, Lukasz Jaromin, Edmund Jay, Michael B. Jones, Tom Jones, Judith Kahrer, Takahiko Kawasaki, Niels Klomp, Ronald Koenig, Micha Kraus, Markus Kreusch, Philipp Lehwalder, Adam Lemmon, Dave Longley, David Luna, Daniel McGrogan, Jeremie Miller, Kenichi Nakamura, Rolson Quadras, Nat Sakimura, Sudesh Shetty, Oliver Terbu, Dimitri James Tsiflitzis, Mike Varley, Arjen van Veen, Jan Vereecken, David Waite, Jacob Ward for their valuable feedback and contributions to this specification.

# Notices

Copyright (c) 2024 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that MAY cover technology that MAY be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -15

   * credential response always returns an array when not returning a transaction_id with the option for additional meta-data
   * deferred credential response always returns an array (same as credential response)
   * notification_id is now used for an issuance flow that can contain more than one credential
   * Fixed #375: Enabled non-breaking extensibility.

   -14
   
   * removes CWT proof type
   * removes the Batch Credential Endpoint
   * clarify that authorization_details can be present in the Token Request for Pre-Authorized Code Flow when multiple Credential Configurations are present in the Credential Offer
   * make `credential_identifiers` mandatory for `authorization_details` flow
   * changes proof type descriptions to accommodate for the batch issuance changes
   * fix indentation of examples
   * changes proof type descriptions to accomodate for the batch issuance changes
   * changed Credential Endpoint to enable requesting multiple instances of a particular Credential Configuration and Dataset with different cryptographic material
   * clarify optionality of scope and `authorization_details` for Authorization Request
   * Define Credential Format as a term
   * Define Credential Dataset as a term
   * Define Credential Configuration as a term
   * remove use of the `authorization_pending` and `slow_down` error codes
   * removed `c_nonce` and `c_nonce_expires_in` from the token endpoint response
   * added a Nonce Endpoint where a Client can acquire a fresh c_nonce value without the overhead of a full Credential Request

   -13

   * change the structure of `proof_types` from an array to a `proof_types_supported` map that contains a required `proof_signing_alg_values_supported` parameter
   * renamed `cryptographic_suites_supported` to `credential_signing_alg_values_supported` to clarify the purpose of the parameter
   * renamed `credential_configurations` Credential Offer parameter to  `credential_configuration_ids`
   * remove `format` from the Credential Response
   * added `signed_metadata` parameter
   * clarified that a logo can also be a uri, not limited to just a url
   * moved the annex with Credential format profiles to the top of all annexes
   * added a Notification Endpoint used by the Wallet to notify the Credential Issuer of certain events for issued Credentials
   * completed IANA registrations section
   * clarified description of a `mandatory` claim
   * made sure to use gender-neutral language throughout the specification
   * added an option in `authorization_details` to use `credential_configuration_id` pointing to the name of a `credential_configurations_supported` object in the Credential Issuer's Metadata; in addition to an option to use format and type.
   * renamed `credentials` Credential Offer parameter to `credential_configuration_ids`
   * renamed `credentials_supported` Credential Issuer metadata parameter to `credential_configurations_supported`
   * grouped `credential_encryption_jwk`, `credential_response_encryption_alg` and `credential_response_encryption_enc` from Credential Request into a single `credential_response_encryption` object
   * replaced `user_pin_required` in Credential Offer with a `tx_code` object that also now contains `description` and `length`
   * reworked flow description in Overview section
   * removed Credential Offer examples from Credential format profiles
   * added support for HTTP Accept-Language Header in the request for Credential Issuer Metadata to request a subset for display data
   * clarified how the Credential Issuer indicates that it requires proof of possession of the cryptographic key material in the Credential Request
   * added an option to use data integrity proofs as proof of possession of the cryptographic key material in the Credential Request
   * added privacy considerations
   * clarified that AS that only supports pre-auth grant can omit `response_types_supported` metadata
   * added `background_image` credential issuer metadata
   * editorial clean-up (fix capitalization, etc.)

   -12

   * changed `authorization_servers` Credential Issuer metadata parameter to be an array.
   * changed the structure of the `credentials_supported` parameter to a map from array of objects
   * changed the structure of `credentials` parameter in Credential Offer to only be a string (no more objects) whose value is a key in the `credentials_supported` map   
   * added an option to return `credential_identifiers` in `authorization_details` Token Response parameter that can be used to identify Credentials with the same metadata but different claimset/claim values and/or simplify the Credential request even when only one Credential is being issued.
   * clarified that credential offer cannot be signed
   * clarified that Credential error response can be authorization (rfc6750) or Credential request error
   * renamed proof to key proof and added key proof replay security considerations
   * aligned deferred authorization with RFC 8628 and CIBA
   * changed Deferred Endpoint to require same access tokens as (batch) Credential endpoint(s)
   * renamed acceptance_token to transaction_id and changed it to a request parameter, and clarified that HTTP status 202 should be returnedin case of deferred issuance
   * added Deferred Endpoint error response section 
   * added Deferred Endpoint metadata
   * added CWT proof type
   * editorial clean-up (remove indefinite articles and duplicates, etc.)

   -11

   * editorial changes to improve readability

   -10

   * introduced differentiation between Credential Issuer and Authorization Server 
   * relaxed Client identification requirements for Pre-Authorized Code Grant Type
   * renamed issuance initiation endpoint to Credential Offer Endpoint
   * added `grants` structure to Credential offer

   -09

   * reworked Credential type identification and issuer metadata
   * changed format of issuer initiated Credential Request to JSON
   * added option to include Credential data by reference in issuer initiated Credential Request
   * added profiles for W3C VCs and ISO mDL
   * added Batch Credential Endpoint

   -08

   * reworked use of OAuth 2.0 scopes to be more flexible for implementers
   * added text on scope related error handling
   * changed media type of a Credential Request to application/json from application/x-www-form-urlencoded
   
   -07

   * restructured the entire specification as following:
   * reorganized defined parameters and mechanisms around endpoints, not the flows
   * merged requirements section into a simpler overview section
   * moved metadata to the after describing the endpoints
   * broke down one large diagram into two simpler ones

   -06

   * added issuer metadata
   * made Credential Response more flexible regarding Credential encoding 
   * changed file name to match specification name
   * renamed specification to reflect OAuth 2.0 being the base protocol

   -05

   * added support for Pre-Authorized Code Flow
   * changed base protocol to OAuth 2.0

   -04

   * added support for requesting Credential authorization with scopes 
   * removed support to pass VPs in the Authorization Request
   * reworked "proof" parameter definition and added "jwt" proof type

   -03

   * added issuance initiation endpoint
   * Applied cleanups suggested by Mike Jones post adoption

   -02

   * Adopted as WG document

   -01

   * Added Request & Response syntax and descriptions
   * Reworked and extended sequence diagram

   -00 

   *  initial revision
