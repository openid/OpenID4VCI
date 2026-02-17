%%%
title = "OpenID for Verifiable Credential Issuance 1.1 - Editor's draft"
abbrev = "openid-4-verifiable-credential-issuance"
ipr = "none"
workgroup = "OpenID Digital Credentials Protocols"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-4-verifiable-credential-issuance-1_1-01"
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

[[author]]
initials="P."
surname="Bastian"
fullname="Paul Bastian"
organization="Bundesdruckerei"
    [author.address]
    email = "paul.bastian@bdr.de"

%%%

.# Abstract

This specification defines an API for the issuance of Verifiable Credentials.

{mainmatter}

# Introduction

This specification defines an OAuth-protected API for the issuance of Verifiable Credentials. Credentials can be of any format, including, but not limited to, IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mdoc [@ISO.18013-5], and W3C VCDM [@VC_DATA].

Verifiable Credentials are very similar to identity assertions, like ID Tokens in OpenID Connect [@OpenID.Core], in that they allow a Credential Issuer to assert End-User claims. A Verifiable Credential follows a pre-defined schema (the Credential type) and MAY be bound to a certain Holder, e.g., through Cryptographic Key Binding. Verifiable Credentials can be securely presented for the End-User to the RP, without the involvement of the Credential Issuer.

Access to this API is authorized using OAuth 2.0 [@!RFC6749], i.e., the Wallet uses OAuth 2.0 to obtain authorization to receive Verifiable Credentials. This way the issuance process can benefit from the proven security, simplicity, and flexibility of OAuth 2.0 and existing OAuth 2.0 deployments and OpenID Connect OPs (see [@OpenID.Core]) can be extended to become Credential Issuers.

## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when, they appear in all capitals, as shown here.

# Terminology {#terminology}

This specification uses the terms "Access Token", "Authorization Endpoint", "Authorization Request", "Authorization Response", "Authorization Code Grant", "Authorization Server", "Client", "Client Authentication", "Client Identifier", "Grant Type", "Refresh Token", "Token Endpoint", "Token Request" and "Token Response" defined by OAuth 2.0 [@!RFC6749], the terms "End-User", "Entity", and "Request Object" as defined by OpenID Connect Core [@!OpenID.Core], the term "JSON Web Token (JWT)" defined by JSON Web Token (JWT) [@!RFC7519], the term "JOSE Header" defined by JSON Web Signature (JWS) [@!RFC7515].

Base64url-encoded denotes the URL-safe base64 encoding without padding defined in Section 2 of [@!RFC7515].

This specification also defines the following terms. In the case where a term has a definition that differs, the definition below is authoritative for this specification.

Credential Dataset:
:  A set of one or more claims about a subject, provided by a Credential Issuer.

Credential (or Verifiable Credential (VC)):
:  An instance of a Credential Configuration with a particular Credential Dataset, that is signed by an Issuer and can be cryptographically verified. An Issuer may provide multiple Credentials as separate instances of the same Credential Configuration and Credential Dataset but with different cryptographic values. In this specification, the term "Verifiable Credential" is also referred to as "Credential". It's important to note that the use of the term "Credential" here differs from its usage in [@!OpenID.Core] and [@!RFC6749]. In this context, "Credential" specifically does not encompass other meanings such as passwords used for login credentials.

Credential Format:
:  Data Model used to create and represent Credential information. This format defines how various pieces of data within a Verifiable Credential are organized and encoded, ensuring that the Verifiable Credential can be consistently understood, processed, and verified by different systems. The exact parameters required to use a Credential Format in the context of this specification are defined in the Credential Format Profile. Definitions of Credential Formats is out of scope for this specification. Examples for Credential Formats are IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mdoc [@ISO.18013-5], and W3C VCDM [@VC_DATA].

Credential Format Profile:
:  Set of parameters specific to individual Credential Formats. This specification provides Credential Format Profiles for IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mdoc [@ISO.18013-5], and W3C VCDM [@VC_DATA], which can be found in the section (#format-profiles). Additionally, other specifications or deployments can define their own Credential Format Profiles by utilizing the extension points defined in this specification.

Credential Format Identifier:
:  An identifier to denote a specific Credential Format in the context of this specification. This identifier implies the use of parameters specific to the respective Credential Format Profile.

Credential Configuration:
: Credential Issuer's description of a particular kind of Credential that the Credential Issuer is offering to issue, along with metadata pertaining to the issuance process and the issued Credentials. Each Credential Configuration references a Credential Format and specifies the corresponding parameters given in the Credential Format Profile. It also includes information about how issuance of a described Credential be requested, information on cryptographic methods and algorithms supported for issuance, and display information to be used by the Wallet. A Credential Configuration is identified by a Credential Configuration Identifier string that is unique to an Issuer. Credential Issuer metadata includes one or more Credential Configurations.

Presentation:
: Data that is presented to a specific Verifier, derived from one or more Verifiable Credentials that can be from the same or different Credential Issuers. It can be of any Credential Format.

Credential Issuer (or Issuer):
:  An entity that issues Verifiable Credentials. In the context of this specification, the Credential Issuer acts as an OAuth 2.0 Resource Server (see [@!RFC6749]). The Credential Issuer might also act as an Authorization Server.

Holder:
:  An entity that receives Verifiable Credentials and has control over them to present them to the Verifiers as Presentations.

Verifier:
:  An entity that requests, receives, and validates Presentations.

Issuer-Holder-Verifier Model:
:  Model that facilitates the exchange of claims, where claims are issued as Verifiable Credentials independently of the process of presenting them to Verifiers in the form of Presentations. An issued Verifiable Credential may be used multiple times, although this is not a requirement.

Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential.

Cryptographic Holder Binding or Cryptographic Key Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proving control over the same private key during the issuance and presentation. The concrete mechanism depends on the Credential Format. For example, in IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], the Issuer can enable this binding by including a public key or a reference to a public key that matches to the private key controlled by the Holder.

Claims-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Credential by proving certain claims, e.g., name and date of birth, for example by presenting another Credential. Claims-based Holder Binding allows long-term, cross-device use of a Credential as it does not depend on cryptographic key material stored on a certain device. One example of such a Credential could be a diploma.


Wallet:
:  An entity used by the Holder to request, receive, store, present, and manage Verifiable Credentials and cryptographic key material. There is no single deployment model of a Wallet: Credentials and keys can be stored and managed either locally, through a remote self-hosted service, or via a remote third-party service. In the context of this specification, the Wallet acts as an OAuth 2.0 Client (see [@!RFC6749]) and obtains an Access Token to access an OAuth 2.0 Resource Server (Credential Endpoint).

Deferred Credential Issuance:
:  Issuance of Credentials not directly in the response to a Credential issuance request but following a period of time that can be used to perform certain offline business processes.

# Overview

## Credential Issuer

This specification defines an API for Credential issuance provided by a Credential Issuer. The API is comprised of the following endpoints:

* A mandatory Credential Endpoint from which Credentials can be issued (see (#credential-endpoint)). From this endpoint, one Credential, or multiple Credentials with the same Credential Format and Credential Dataset can be issued in one request.
* An optional Nonce Endpoint from which a fresh `c_nonce` value can be obtained to be used in proof of possession of key material in a subsequent request to the Credential Endpoint (see (#nonce-endpoint)).
* An optional Deferred Credential Endpoint to allow for the deferred delivery of Credentials (see (#deferred-credential-issuance)).
* An optional mechanism for the Credential Issuer to make a Credential Offer to the Wallet to encourage the Wallet to start the issuance flow (see (#credential-offer-endpoint)).
* An optional mechanism for the Credential Issuer to receive notification(s) from the Wallet about the status of the Credential(s) that have been issued (see (#notification-endpoint)).
* A mechanism for the Credential Issuer to publish metadata about the Credentials it is capable of issuing (see (#credential-issuer-metadata)).

The Credential Endpoint may bind an issued Credential to specific cryptographic key material, in which case Credential requests include proof(s) of possession or attestations for the key material. Multiple key proof types are supported.

## OAuth 2.0

According to the OAuth 2.0 framework, each Credential Issuer acts as a Resource Server that is protected by an Access Token issued by an Authorization Server, as defined in OAuth 2.0 [@!RFC6749]. The same Authorization Server can protect one or more Credential Issuers. Wallets identify the Authorization Server for a Credential Issuer by referring to the Credential Issuer's metadata (see (#credential-issuer-metadata)).

All OAuth 2.0 Grant Types and extensions mechanisms can be used in conjunction with the Credential issuance API. Aspects not defined in this specification are expected to follow [@!RFC6749]. 

Existing OAuth 2.0 mechanisms are extended as follows:

* A new Grant Type "Pre-Authorized Code" is defined to facilitate flows where the preparation of the Credential issuance is conducted before the actual OAuth flow starts (see (#pre-authz-code-flow)).
* A new authorization details [@!RFC9396] type `openid_credential` is defined to convey the details about the Credentials (including Credential Dataset, Credential Formats, and Credential types) the Wallet wants to obtain (see (#authorization-details)).
* Client metadata is used to convey the Wallet's metadata. The new Client metadata parameter `credential_offer_endpoint` is added to allow a Wallet (acting as OAuth 2.0 client) to publish its Credential Offer Endpoint (see (#client-metadata)).
* Authorization Endpoint: The additional parameter `issuer_state` is added to convey state in the context of processing an issuer-initiated Credential Offer (see (#credential-authz-request)).

## Core Concepts

In the context of this specification, Credential Datasets, Credential Format and Credential Format Profile are defined in (#terminology).
While in principle independent of each other, the Credential Dataset and the Credential Format can have a relationship in the sense that an Issuer may only offer certain Credential Formats for certain Credential Datasets.

An End-User typically authorizes the issuance of Credentials with a specific Credential Dataset, but does not usually care about the Credential Format. The same Credential Dataset may even be issued in different Credential Formats or with multiple Credential instances.

### Credential Formats and Credential Format Profiles

This specification is Credential Format agnostic and allows implementers to leverage specific capabilities of Credential Formats of their choice.
To this end, extension points to add Credential Format specific parameters in the Credential Issuer metadata, Credential Offer, Authorization Request, and Credential Request are defined.

Credential Format Profiles for IETF SD-JWT VC [@I-D.ietf-oauth-sd-jwt-vc], ISO mdoc [@ISO.18013-5], and W3C VCDM [@VC_DATA] are specified in (#format-profiles).
Other specifications or deployments can define their own Credential Format Profiles using the above-mentioned extension points.

### Batch Credential Issuance

This specification enables the issuance of Verifiable Credentials through the Credential Endpoint. 
A single request to this endpoint may request the issuance of a batch of one or more Verifiable Credentials.

Credentials can vary in their format, including Credential Format Profile-specific parameters, in their contents known as the Credential Dataset, and in the cryptographic data such as Issuer signatures, hashes, and keys used for Cryptographic Key Binding.
Credentials can therefore vary in the following dimensions:

- Credential Format
- Credential Dataset
- Cryptographic Data

In the context of a single request, the batch of issued Credentials sent in response MUST share the same Credential Format and Credential Dataset, but SHOULD contain different Cryptographic Data. For example to achieve unlinkability between the Credentials, each credential should be bound to different cryptographic keys.

To issue multiple Verifiable Credentials with differing Credential Formats or Credential Datasets, multiple requests MUST be sent to the Credential Endpoint.

### Issuance Flow Variations

The issuance can have multiple characteristics that can be combined depending on the use cases:

* Authorization Code Flow or Pre-Authorized Code Flow: The Credential Issuer can obtain End-User information to turn into a Verifiable Credential using End-User authentication and consent at the Credential Issuer's Authorization Endpoint (Authorization Code Flow) or using out-of-band mechanisms outside of the issuance flow (Pre-Authorized Code Flow).
* Wallet initiated or Issuer initiated: The request from the Wallet can be sent to the Credential Issuer without any gesture from the Credential Issuer (Wallet Initiated) or following the communication from the Credential Issuer (Issuer Initiated).
* Same-device or Cross-device Credential Offer: The End-User may receive the Credential Offer from the Credential Issuer either on the same device as the device the Wallet resides on, or through any other means, such as another device or postal mail, so that the Credential Offer can be communicated to the Wallet.
* Immediate or Deferred: The Credential Issuer can issue the Credential directly in response to the Credential Request (immediate) or requires time and needs the Wallet to come back to retrieve Credential (deferred).

The following subsections illustrate some of the authorization flows supported by this specification.

### Identifying Credentials Being Issued Throughout the Issuance Flow {#identifying_credential}

Below is the summary of how Credential(s) that are being issued are identified throughout the issuance flow:

- In the Credential Offer, the Credential Issuer identifies offered Credential Configurations
  using the `credential_configuration_ids` parameter.
- When the Wallet uses Authorization Details in the Authorization Request, the Wallet uses
  the `credential_configuration_id` parameter to identify the requested Credential Configurations.
  The Authorization Server returns an `authorization_details` parameter containing
  the `credential_identifiers` parameter in the Token Response,
  and the Wallet uses those `credential_identifier` values in the Credential Request.
- When the Wallet uses `scope` parameter in the Authorization Request, the `scope` value(s)
  are used to identify requested Credential Configurations. In this case, the Authorization Server has two options.
  If the Authorization Server supports returning `credential_identifiers` parameter
  in the Token Response, it MAY do so, in which case the Wallet uses those `credential_identifier` values
  in the Credential Request. If the Authorization Server does not support returning an `authorization_details` parameter containing the
  `credential_identifiers` parameter in the Token Response, the Wallet uses `credential_configuration_id` parameter
  in the Credential Request.


## Authorization Code Flow {#authorization-code-flow}

The Authorization Code Flow uses the grant type `authorization_code` as defined in [@!RFC6749] to issue Access Tokens.

Figure 1 shows the Authorization Code Flows with the two variations that can be implemented for the issuance of Credentials, as outlined in this specification:

* **Wallet-initiated variation**, described in (#use-case-4) or (#use-case-6);
* **Issuer-initiated variation**, described in (#use-case-1).

Please note that the diagram does not illustrate all the optional features defined in this specification.

!---
~~~ ascii-art
+----------+    +-----------+        +----------------------+   +-------------------+
| End-User |    |   Wallet  |        | Authorization Server |   | Credential Issuer |
+----------+    +-----------+        +----------------------+   +-------------------+
      |               |                           |                        |
      | (1a) End-User |                           |                        |
      |  selects      |  (1b) Credential Offer    |                        |
      |  Credential-->|  (credential type)        |                        |
      |               |<---------------------------------------------------|
      |               |                           |                        |
      |               |  (2) Obtains Issuer's     |                        |
      |               |      Credential Issuer    |                        |
      |               |      metadata             |                        |
      |               |--------------------------------------------------->|
      |               |                           |                        |
      |               |                           |  (3) Authorization     |
      |               |                           |      Request           |
      |               |                           |      (type(s) of       |
      |               |                           |      Credentials to    |
      |               |                           |      be issued)        |
      |               |-------------------------->|                        |
      |               |                           |                        |
      |  End-User Authentication / Consent        |                        |
      |               |                           |  (4) Authorization     |
      |               |                           |      Response (code)   |
      |               |<--------------------------|                        |
      |               |                           |                        |
      |               |                           |  (5) Token Request     |
      |               |                           |      (code)            |
      |               |-------------------------->|      Token Response    |
      |               |                           |      (Access Token)    |
      |               |<--------------------------|                        |
      |               |                           |                        |
      |               |  (6) Credential Request   |                        |
      |               |      (Access Token,       |                        |
      |               |       proof(s))           |                        |
      |               |--------------------------------------------------->|
      |               |                           |                        |
      |               |      Credential Response  |                        |
      |               |      with Credential(s)   |                        |
      |               |      OR Transaction ID    |                        |
      |               |<---------------------------------------------------|
~~~
!---
Figure: Issuance using Authorization Code Flow 

(1a) The Wallet-initiated flow begins as the End-User requests a Credential via the Wallet from the Credential Issuer. The End-User either selects a Credential from a pre-configured list of Credentials ready to be issued, or alternatively, the Wallet gives guidance to the End-User to select a Credential from a Credential Issuer based on the information it received in the presentation request from a Verifier.

(1b) The Issuer-initiated flow begins as the Credential Issuer generates a Credential Offer for certain Credential(s) that it communicates to the Wallet, for example, as a QR code or as a URI. The Credential Offer contains the Credential Issuer's URL and the information about the Credential(s) being offered. This step is defined in (#credential-offer).

(2) The Wallet uses the Credential Issuer's URL to fetch the Credential Issuer metadata, as described in (#credential-issuer-metadata). The Wallet needs the metadata to learn the Credential types and formats that the Credential Issuer supports and to determine the Authorization Endpoint (OAuth 2.0 Authorization Server) as well as Credential Endpoint required to start the request. This specification supports configurations where the Credential Endpoint and the Authorization Endpoint are managed by either separate entities or a single entity.

(3) The Wallet sends an Authorization Request to the Authorization Endpoint. The Authorization Endpoint processes the Authorization Request, which typically includes authenticating the End-User and gathering End-User consent. Note: The Authorization Request may be sent as a Pushed Authorization Request.

(4) The Authorization Endpoint returns the Authorization Response with the Authorization Code upon successfully processing the Authorization Request.

Note: Steps (3) and (4) may happen in the front channel, by redirecting the End-User via the User Agent. Those steps are defined in (#authorization-endpoint). The Authorization Server and the User Agent may exchange any further messages between the steps if required by the Authorization Server to authenticate the End-User. The Authorization Server may request Credential presentation as a means to authenticate or identify the End-User during the issuance flow, as described in (#use-case-5), by using the mechanism defined in (#interactive-authorization-endpoint).

(5) The Wallet sends a Token Request to the Token Endpoint with the Authorization Code obtained in Step (4). The Token Endpoint returns an Access Token in the Token Response upon successfully validating the Authorization Code. This step happens in the back-channel communication (direct communication between two systems using HTTP requests and responses without using redirects through an intermediary such as a browser). This step is defined in (#token-endpoint).

(6) The Wallet sends a Credential Request to the Credential Issuer's Credential Endpoint with the Access Token and (optionally) the proof(s) of possession of the private key of a key pair to which the Credential Issuer should bind the issued Credential to. Upon successfully validating Access Token and proof(s), the Credential Issuer returns a Credential in the Credential Response. This step is defined in (#credential-endpoint).

If the Credential Issuer requires more time to issue a Credential, the Credential Issuer may return a Transaction ID and a time interval in the Credential Response. The Wallet may send a Deferred Credential Request with the Transaction ID to obtain a Credential after the specified time interval has passed, as defined in (#deferred-credential-issuance).

Note: This flow is based on OAuth 2.0 and the Authorization Code Grant type, but this specification can be used with other OAuth 2.0 grant types as well.

## Pre-Authorized Code Flow {#pre-authz-code-flow}

Figure 2 is a diagram of a Credential issuance using the Pre-Authorized Code Flow. In this flow, before initiating the flow with the Wallet, the Credential Issuer first conducts the steps required to prepare for Credential issuance, e.g., End-User authentication and authorization. Consequently, the Pre-Authorized Code is sent by the Credential Issuer to the Wallet. This flow does not use the Authorization Endpoint, and the Wallet exchanges the Pre-Authorized Code for the Access Token directly at the Token Endpoint. The Access Token is then used to request Credential issuance at the Credential Endpoint. See (#use-case-2) for the description of such a use case.

How the End-User provides information required for the issuance of a requested Credential to the Credential Issuer and the business processes conducted by the Credential Issuer to prepare a Credential are out of scope of this specification.

This flow uses the OAuth 2.0 Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`, which is defined in (#credential-offer-parameters).

The following diagram is based on the Credential Issuer-initiated flow, as described in the use case in (#use-case-2). Please note that it does not illustrate all the optional features outlined in this specification.

!---
~~~ ascii-art
+----------+   +-----------+           +----------------------+   +-------------------+
| End-User |   |   Wallet  |           | Authorization Server |   | Credential Issuer |
+----------+   +-----------+           +----------------------+   +-------------------+
      |              |                              |                       |
      |              |  (1) End-User provides       |                       |
      |              |      information required    |                       |
      |              |      for the issuance of     |                       |
      |              |      a certain Credential    |                       |
      |              |----------------------------------------------------->|
      |              |                              |                       |
      |              |  (2) Credential Offer        |                       |
      |              |      (Pre-Authorized Code)   |                       |
      |              |<-----------------------------------------------------|
      |              |  (3) Obtains Issuer's        |                       |
      |              |      Credential Issuer       |                       |
      |              |      metadata                |                       |
      |              |----------------------------------------------------->|
      |   interacts  |                              |                       |
      |------------->|                              |                       |
      |              |                              |                       |
      |              |  (4) Token Request           |                       |
      |              |      (Pre-Authorized Code,   |                       |
      |              |       tx_code)               |                       |
      |              |----------------------------->|                       |
      |              |      Token Response          |                       |
      |              |      (access_token)          |                       |
      |              |<-----------------------------|                       |
      |              |                              |                       |
      |              |  (5) Credential Request      |                       |
      |              |      (access_token, proof(s))|                       |
      |              |----------------------------------------------------->|
      |              |      Credential Response     |                       |
      |              |      (Credential(s))         |                       |
      |              |<-----------------------------------------------------|
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

Credential Issuers MAY also communicate Credential Offers directly to a Wallet's backend, but any mechanism for doing so is currently outside the scope of this specification.

The Credential Offer object, which is a JSON-encoded object with the Credential Offer parameters, can be sent by value or by reference.

The Credential Offer contains a single URI query parameter, either `credential_offer` or `credential_offer_uri`:

* `credential_offer`: Object with the Credential Offer parameters. This MUST NOT be present when the `credential_offer_uri` parameter is present.
* `credential_offer_uri`: String that is a URL using the `https` scheme referencing a resource containing a JSON object with the Credential Offer parameters. This MUST NOT be present when the `credential_offer` parameter is present.

For security considerations, see (#credential-offer-security).

### Credential Offer Parameters {#credential-offer-parameters}

This specification defines the following parameters for the JSON-encoded Credential Offer object:

* `credential_issuer`: REQUIRED. The URL of the Credential Issuer, as defined in (#credential-issuer-identifier), from which the Wallet is requested to obtain one or more Credentials. The Wallet uses it to obtain the Credential Issuer's Metadata following the steps defined in (#credential-issuer-wellknown).
* `credential_configuration_ids`: REQUIRED. A non-empty array of unique strings that each identify one of the keys in the name/value pairs stored in the `credential_configurations_supported` Credential Issuer metadata. The Wallet uses these string values to obtain the respective object that contains information about the Credential being offered as defined in (#credential-issuer-parameters). For example, these string values can be used to obtain `scope` values to be used in the Authorization Request.
* `grants`: OPTIONAL. Object indicating to the Wallet the Grant Types the Credential Issuer's Authorization Server is prepared to process for this Credential Offer. Every grant is represented by a name/value pair. The name is the Grant Type identifier; the value is an object that contains parameters either determining the way the Wallet MUST use the particular grant and/or parameters the Wallet MUST send with the respective request(s). If `grants` is not present or is empty, the Wallet MUST determine the Grant Types the Credential Issuer's Authorization Server supports using the respective metadata. When multiple grants are present, it is at the Wallet's discretion which one to use.

Additional Credential Offer parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

The following values are defined by this specification: 

* Grant Type `authorization_code`:
  * `issuer_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the Wallet that is used to bind the subsequent Authorization Request with a context set up during previous process steps. If the Wallet decides to use the Authorization Code Flow and received a value for this parameter, it MUST include it in the subsequent Authorization Request to the Authorization Server as the `issuer_state` parameter value.
  * `authorization_server`: OPTIONAL string that the Wallet can use to identify the Authorization Server to use with this grant type when `authorization_servers` parameter in the Credential Issuer metadata has multiple entries. It MUST NOT be used otherwise. The value of this parameter MUST match with one of the values in the `authorization_servers` array obtained from the Credential Issuer metadata.
* Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`:
  * `pre-authorized_code`: REQUIRED. The code representing the Credential Issuer's authorization for the Wallet to obtain Credentials of a certain type. This code MUST be short lived and single use. If the Wallet decides to use the Pre-Authorized Code Flow, this parameter value MUST be included in the subsequent Token Request with the Pre-Authorized Code Flow.
  * `tx_code`: OPTIONAL. Object indicating that a Transaction Code is required if present, even if empty. It describes the requirements for a Transaction Code, which the Authorization Server expects the End-User to present along with the Token Request in a Pre-Authorized Code Flow. If the Authorization Server does not expect a Transaction Code, this object is absent; this is the default. The Transaction Code is intended to bind the Pre-Authorized Code to a certain transaction to prevent replay of this code by an attacker that, for example, scanned the QR code while standing behind the legitimate End-User. It is RECOMMENDED to send the Transaction Code via a separate channel. If the Wallet decides to use the Pre-Authorized Code Flow, the Transaction Code value MUST be sent in the `tx_code` parameter with the respective Token Request as defined in (#token-request). If no `length`, `description`, or `input_mode` is given, this object MAY be empty.
    * `input_mode` : OPTIONAL. String specifying the input character set. Possible values are `numeric` (only digits) and `text` (any characters). The default is `numeric`.
    * `length`: OPTIONAL. Integer specifying the length of the Transaction Code. This helps the Wallet to render the input screen and improve the user experience.
    * `description`: OPTIONAL. String containing guidance for the Holder of the Wallet on how to obtain the Transaction Code, e.g., describing over which communication channel it is delivered. The Wallet is RECOMMENDED to display this description next to the Transaction Code input screen to improve the user experience. The length of the string MUST NOT exceed 300 characters. The `description` does not support internationalization, however the Issuer MAY detect the Holder's language by previous communication or an HTTP Accept-Language header within an HTTP GET request for a Credential Offer URI.
  * `authorization_server`: OPTIONAL string that the Wallet can use to identify the Authorization Server to use with this grant type when `authorization_servers` parameter in the Credential Issuer metadata has multiple entries. It MUST NOT be used otherwise. The value of this parameter MUST match with one of the values in the `authorization_servers` array obtained from the Credential Issuer metadata.
  
The following non-normative example shows a Credential Offer object where the Credential Issuer can offer the issuance of two different Credentials (which may even be of different formats):

<{{examples/credential_offer_multiple_credentials.json}}

### Sending Credential Offer by Value Using `credential_offer` Parameter

Below is a non-normative example of a Credential Offer passed by value (with line breaks within values for display purposes only):

```
GET /credential_offer?
  credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.exam
  ple.com%22,%22credential_configuration_ids%22:%5B%22UniversityDegree_JWT%22,%
  22org.iso.18013.5.1.mDL%22%5D,%22grants%22:%7B%22urn:ietf:params:oauth:grant-
  type:pre-authorized_code%22:%7B%22pre-authorized_code%22:%22oaKazRN8I0IbtZ0C7
  JuMn5%22,%22tx_code%22:%7B%7D%7D%7D%7D
```

The following is a non-normative example of a Credential Offer that can be included in a QR code or a link used to invoke a Wallet deployed as a native app (with line breaks within values for display purposes only):

```
openid-credential-offer://?
  credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.exam
  ple.com%22,%22credential_configuration_ids%22:%5B%22org.iso.18013.5.1.mDL%22%
  5D,%22grants%22:%7B%22urn:ietf:params:oauth:grant-type:pre-authorized_code%22
  :%7B%22pre-authorized_code%22:%22oaKazRN8I0IbtZ0C7JuMn5%22,%22tx_code%22:%7B%
  22input_mode%22:%22text%22,%22description%22:%22Please%20enter%20the%20serial
  %20number%20of%20your%20physical%20drivers%20license%22%7D%7D%7D%7D
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

Authorization is obtained using the `authorization_code` grant type defined in [@!RFC6749], optionally with either PAR ((#pushed-authorization-request)) or the Interactive Authorization Endpoint (#interactive-authorization-endpoint). Implementers SHOULD follow the best current practices for OAuth 2.0 Security given in [@!BCP240], see (#securitybcp). This includes the use of mechanisms like PKCE to prevent authorization code interception attacks and Pushed (or Interactive) Authorization Requests to ensure the integrity and authenticity of the authorization request.

## Authorization Request {#credential-authz-request}

An Authorization Request is an OAuth 2.0 Authorization Request as defined in Section 4.1.1 of [@!RFC6749], which requests that access be granted to the Credential Endpoint, as defined in (#credential-endpoint).

There are two possible methods for requesting the issuance of a specific Credential type in an Authorization Request. The first method involves using the `authorization_details` request parameter, as defined in [@!RFC9396], containing one or more authorization details of type `openid_credential`, as specified in (#authorization-details). The second method utilizes scopes, as outlined in (#credential-request-using-type-specific-scope).

See (#identifying_credential) for the summary of the options how requested Credential(s) are identified throughout the Issuance flow.

### Using Authorization Details Parameter {#authorization-details}

Credential Issuers MAY support requesting authorization to issue a Credential using the `authorization_details` parameter.

The request parameter `authorization_details` defined in Section 2 of [@!RFC9396] MUST be used to convey the details about the Credentials the Wallet wants to obtain. This specification introduces a new authorization details type `openid_credential` and defines the following parameters to be used with this authorization details type:

* `type`: REQUIRED. String that determines the authorization details type. It MUST be set to `openid_credential` for the purpose of this specification.
* `credential_configuration_id`: REQUIRED. String specifying a unique identifier of the Credential being described in the `credential_configurations_supported` map in the Credential Issuer Metadata as defined in (#credential-issuer-parameters). The referenced object in the `credential_configurations_supported` map conveys the details of the requested Credential, such as the format and format-specific parameters like `vct` for SD-JWT VC or `doctype` for ISO mdoc. This specification defines those Credential Format specific Issuer Metadata in (#format-profiles).
* `claims`: OPTIONAL. A non-empty array of claims description objects as defined in (#claims-description-authorization-details).

Additional `authorization_details` data fields MAY be defined and used when the `type` value is `openid_credential`.
Note that this effectively defines an authorization details type that is never considered invalid due to unknown fields.

The following is a non-normative example of an `authorization_details` object with a `credential_configuration_id`:

<{{examples/authorization_details.json}}

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
  &redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
  
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
  &redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
Host: server.example.com
```

If a scope value related to Credential issuance and the `authorization_details` request parameter containing objects of type `openid_credential` are both present in a single request, the Credential Issuer MUST interpret these individually. However, if both request the same Credential type, then the Credential Issuer MUST follow the request as given by the authorization details object.

### Additional Request Parameters {#additional-request-parameters}

This specification defines the following request parameter that can be supplied in an Authorization Request:

* `issuer_state`: OPTIONAL. String value identifying a certain processing context at the Credential Issuer. A value for this parameter is typically passed in a Credential Offer from the Credential Issuer to the Wallet (see (#credential-offer)). This request parameter is used to pass the `issuer_state` value back to the Credential Issuer.

Note: When processing the Authorization Request, the Credential Issuer MUST take into account that the `issuer_state` is not guaranteed to originate from this Credential Issuer in all circumstances. It could have been injected by an attacker.

Additional Authorization Request parameters MAY be defined and used,
as described in [@!RFC6749].
The Authorization Server MUST ignore any unrecognized parameters.

### Pushed Authorization Request {#pushed-authorization-request}

Use of Pushed Authorization Requests or the Interactive Authorization Endpoint (see (#interactive-authorization-endpoint) is RECOMMENDED to ensure confidentiality, integrity, and authenticity of the request data and to avoid issues caused by large requests sizes.

Below is a non-normative example of a Pushed Authorization Request:

```
POST /par HTTP/1.1
Host: server.example.com
OAuth-Client-Attestation: eyJ...
OAuth-Client-Attestation-PoP: eyJ...
Content-Type: application/x-www-form-urlencoded

response_type=code
&client_id=CLIENT1234
&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
&code_challenge_method=S256
&redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
&authorization_details=...
```

Below is a non-normative example of a response to a successful request:

```
HTTP/1.1 201 Created
Content-Type: application/json
Cache-Control: no-cache, no-store

{
  "request_uri": "urn:ietf:params:oauth:request_uri:6esc_11ACC5bwc014ltc14eY22c",
  "expires_in": 60
}
```

Below is a non-normative example of the GET request that might subsequently be sent by the Browser:

```
GET /authorize?client_id=s6BhdRkqt3
  &request_uri=urn%3Aietf%3Aparams%3Aoauth%3Arequest_uri%3A6esc_11ACC5bwc014ltc14eY22c
Host: server.example.com
```

## Successful Authorization Response {#authorization-response}

Authorization Responses MUST be made as defined in [@!RFC6749].

Below is a non-normative example of a successful Authorization Response:

```
HTTP/1.1 302 Found
Location: https://wallet.example.org/cb?
  code=SplxlOBeZQQYbYS6WxSbIA
```

## Authorization Error Response

The Authorization Error Response MUST be made as defined in [@!RFC6749].

Below is a non-normative example of an unsuccessful Authorization Response:

```
HTTP/1.1 302 Found
Location: https://wallet.example.org/cb?
  error=invalid_request
  &error_description=Unsupported%20response_type%20value
```

# Interactive Authorization Endpoint {#interactive-authorization-endpoint}

This is an extension of the traditional Authorization Endpoint defined in [@!RFC6749], enabling complex authentication and authorization flows where interaction occurs directly with the Wallet rather than being intermediated by a browser.
A primary use case is requiring the Presentation of a Credential as a prerequisite for issuing a new Credential.
Support for the Interactive Authorization Endpoint is OPTIONAL.

The Authorization Server indicates support for interactive authorization by publishing the `interactive_authorization_endpoint` parameter in its Authorization Server Metadata as defined in (#as-metadata).

The following figure illustrates a flow using the Interactive Authorization Endpoint, where the Authorization Server requests a Presentation (of another Credential) from the Wallet as part of the authorization process to issue a Credential to that Wallet. The exact deployment model of the OpenID4VP Verifier in relation to the Authorization Server is out of scope of this specification. It can be integrated into the Authorization Server or a separate component, in which case backchannel communication between the Verifier and Authorization Server would need to happen (not shown here).


!---
~~~ ascii-art

 +-----------+            +----------------------+     +--------------------+
 |   Wallet  |            | Authorization Server |     | Credential Issuer  |
 +-----------+            +----------------------+     +--------------------+
       |                              |                        |
       |                              |                        |
       |----------------------------->|  (1) Interactive       |
       |                              |      Authorization     |
       |                              |      Request           |
       |                              |                        |
       |                              |                        |
       |<-----------------------------|  (2) Interactive       |
       |                              |      Authorization     |
       |                              |      Response          |
       |                              |      (presentation     |
       |                              |      request,          |
       |                              |      auth_session)     |
       |                              |                        |
       |                              |                        |
       |----------------------------->|  (3) Interactive       |
       |                              |      Authorization     |
       |                              |      Request           |
       |                              |      (auth_session,    |
       |                              |      presentation      |
       |                              |      response)         |
       |                              |                        |
       |<-----------------------------|  (4) Interactive       |
       |                              |      Authorization     |
       |                              |      Response (code)   |
       |                              |                        |
       |----------------------------->|  (5) Token Request     |
       |                              |      (code)            |
       |                              |                        |
       |<-----------------------------|  (6) Token Response    |
       |                              |      (Access Token)    |
       |                              |                        |
       |                              |                        |
       |  (7) Credential Request      |                        |
       |      (Access Token, proof(s))|                        |
       |------------------------------------------------------>|
       |                              |                        |
       |  (8) Credential Response     |                        |
       |       with Credential(s) OR  |                        |
       |       Transaction ID         |                        |
       |<------------------------------------------------------|
~~~
!---
Figure: Issuance using the Interactive Authorization Endpoint

## Interactive Authorization Request {#interactive-authorization-request}

All communication with the Interactive Authorization Endpoint MUST utilize TLS.

The rules for client authentication as defined in [@!RFC9126] and [@!RFC6749] for pushed authorization requests, including the applicable authentication methods, apply for all requests to the Interactive Authorization Endpoint as well.

Note: In case a Wallet Attestation is required by the Authorization Server, it has to be included in this request.

### Initial Request

The initial request to the Interactive Authorization Endpoint is formed and sent in the same way as PAR request as defined in Section 2.1 of [@!RFC9126]. The contents of the request are the same as in a regular Authorization Request as defined in (#credential-authz-request), with the following addition:

`interaction_types_supported`: REQUIRED. Comma-separated list of strings indicating the types of interactions that the Wallet supports. The order of the values is not significant. The following values are defined by this specification:

* `openid4vp_presentation`: Indicates that the Wallet supports an OpenID4VP Presentation interaction, as defined in (#iae-require-presentation).
* `redirect_to_web`: Indicates that the Wallet supports a redirect to a web-based interaction, as defined in (#iae-redirect-to-web).

Custom interaction types (see (#iae-custom-extensions)) MAY be defined by the Authorization Server and used in the `interaction_types_supported` parameter.

When the wallet includes `redirect_to_web` in `interaction_types_supported`, the `code_challenge` and `code_challenge_method` parameters (see (#securitybcp)) are included in the initial request.

The following non-normative example shows an initial request to the Interactive Authorization Endpoint:

```http
POST /iae HTTP/1.1
Host: server.example.com
OAuth-Client-Attestation: eyJ...
OAuth-Client-Attestation-PoP: eyJ...
Content-Type: application/x-www-form-urlencoded

response_type=code
&client_id=CLIENT1234
&code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
&code_challenge_method=S256
&redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
&authorization_details=...
&interaction_types_supported=openid4vp_presentation%2Credirect_to_web
```

The following non-normative example shows an initial request to the Interactive Authorization Endpoint with a signed request object:

```http
POST /iae HTTP/1.1
Host: server.example.com
OAuth-Client-Attestation: eyJ...
OAuth-Client-Attestation-PoP: eyJ...
Content-Type: application/x-www-form-urlencoded

request=eyJrd...
```

The following non-normative example shows a payload of a signed request object:

```json
{
  "iss": "CLIENT1234",
  "aud": "https://server.example.com",
  "response_type": "code",
  "client_id": "CLIENT1234",
  "code_challenge": "E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM",
  "code_challenge_method": "S256",
  "redirect_uri": "https%3A%2F%2Fclient.example.org%2Fcb",
  "authorization_details": [
    {
      "type": "openid_credential",
      "credential_configuration_id": "UniversityDegreeCredential"
    }
  ],
  "interaction_types_supported": "openid4vp_presentation,redirect_to_web"
}
```

### Follow-up Request {#follow-up-request}

Follow-up requests to the Interactive Authorization Endpoint MUST include the `auth_session` value received most recently from the Authorization Server (see (#iae-interaction-required-response)).

Besides `auth_session`, follow-up requests only include the parameters that are in response to the interaction type the Authorization Server requested in the most recent response. The specific parameters are defined by each interaction type.

The following non-normative example shows a follow-up request to the Interactive Authorization Endpoint where the Wallet has already received an `auth_session`:

```http
POST /iae HTTP/1.1
Host: server.example.com
OAuth-Client-Attestation: eyJ...
OAuth-Client-Attestation-PoP: eyJ...
Content-Type: application/x-www-form-urlencoded

auth_session=wxroVrBY2MCq4dDNGXACS
```

## Interactive Authorization Response

Upon receiving an Interactive Authorization Request, the Authorization Server determines whether the Authorization Request is syntactically and semantically correct and whether the information provided by the Wallet so far is sufficient to grant authorization for the Credential issuance.
The response to an Interactive Authorization Request is an HTTP message with the content type `application/json` and a JSON document in the body that indicates either

 1. that user interaction is required, either a Presentation or a custom interaction, as defined in (#iae-interaction-required-response), or
 2. a successful completion of the authorization, as defined in (#iae-authorization-code-response), or
 3. an error as defined in Section 2.3 of [@!RFC9126] including the additional error codes defined in (#iae-error-response).

Except in error cases, the following key is required in the JSON document of the response:

* `status`: REQUIRED. String indicating whether an additional interaction is required or the authorization has been completed.

Depending on this assessment, the response from the Interactive Authorization Endpoint can take one of the following forms:

* Interaction Required Response as defined in (#iae-interaction-required-response)
* Authorization Code Response as defined in (#iae-authorization-code-response)
* Interactive Authorization Error Response as defined in (#iae-error-response)

### Interaction Required Response {#iae-interaction-required-response}

By setting `status` to `require_interaction` in the response, the Authorization Server requests an additional user interaction.
In this case, the following keys MUST be present in the response as well:

* `type`: REQUIRED. String indicating which type of interaction is required, as defined below. The Authorization Server MUST NOT set this to a value that was not included in the `interaction_types_supported` parameter sent by the Wallet.
* `auth_session`: REQUIRED. String containing a value that allows the Authorization Server to associate subsequent requests by this Wallet with the ongoing authorization request sequence. Wallets SHOULD treat this value as an opaque value. The value returned MUST be distinct for each interactive authorization response.

The Wallet MUST include the most recently received `auth_session` in follow-up requests to the Interactive Authorization Endpoint.

If a Wallet receives a `type` value that it does not recognize, it MUST abort the issuance process.

Additional keys are defined based on the type of interaction, as shown next.

#### Require Presentation {#iae-require-presentation}

If `type` is set to `openid4vp_presentation`, as shown in the following example, the response MUST further include an `openid4vp_request` parameter containing an OpenID4VP Authorization Request. The contents of the request is the same as for requests passed to the Digital Credentials API (see Appendix A.2 and Appendix A.3 of [@!OpenID4VP]), except as follows:

* The `response_mode` MUST be either `iae_post` for unencrypted responses or `iae_post.jwt` for encrypted responses. These modes are used to indicate to the Wallet to return the response back to the same Interactive Authorization Endpoint.
* If `expected_origins` is present, it MUST contain only the derived Origin of the Interactive Authorization Endpoint as defined in Section 4 in [@RFC6454]. For example, the derived Origin from `https://example.com/iae` is `https://example.com`.

The following is a non-normative example of an unsigned Authorization Request:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "status": "require_interaction",
  "type": "openid4vp_presentation",
  "auth_session": "wxroVrBY2MCq4dDNGXACS",
  "openid4vp_request": {
    "response_type": "vp_token",
    "response_mode": "iae_post",
    "dcql_query": {
      "credentials": [
        {
          "id": "some_identity_credential",
          "format": "dc+sd-jwt",
          "meta": {
            "vct_values": [ "https://credentials.example.com/identity_credential" ]
          },
          "claims": [
              {"path": ["last_name"]},
              {"path": ["first_name"]}
          ]
        }
      ]
    },
    "nonce": "n-0S6_WzA2Mj"
  }
}
```

The following is a non-normative example of a signed Authorization Request:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "status": "require_interaction",
  "type": "openid4vp_presentation",
  "auth_session": "wxroVrBY2MCq4dDNGXACS",
  "openid4vp_request": {
    "request": "eyJhbGciOiJF..."
  }
}
```

The Wallet MUST process the Authorization Request contained in the `openid4vp_request` parameter as defined in [@!OpenID4VP] to perform a Credential Presentation to the Authorization Server.

For the requested Presentation, the Issuer is acting as a Verifier to the Wallet.
The exact architecture and the deployment of the Issuer's OpenID4VP Verifier is out of scope of this specification.

When processing the request the following logic applies:

  1. If `expected_origins` is present, the Wallet MUST ensure that `expected_origins` contains the derived Origin as defined above.
  2. If the response contains Verifiable Presentations that include Holder Binding, the audience of each of those MUST be properly bound to the Interactive Authorization Endpoint, following the rules defined by their Credential Format. Details on how to do this for each format can be found in the "Interactive Authorization Endpoint Binding" sections under (#format-profiles). Note that the Credential Format here refers to the format of the Verifiable Presentation requested in the OpenID4VP Authorization Request, which may be different from the format used for issuing the Credentials themselves. If any Verifiable Presentation with Holder Binding is not correctly bound to the Interactive Authorization Endpoint, the response MUST be rejected.

The Interactive Authorization Request, which is used to submit the OpenID4VP Authorization Response MUST satisfy the requirements set out in (#follow-up-request). In addition to these requirements, the request MUST also contain the `openid4vp_response` parameter. The value of the `openid4vp_response` parameter is a JSON-encoded object that encodes the OpenID4VP Authorization Response parameters. In the case of an error it instead encodes the Authorization Error Response parameters. When the `response_mode` is `iae_post.jwt` the OpenID4VP Authorization Response MUST be encrypted according to Section 8.3 of [@!OpenID4VP].

The following us an example non-normative example of a Interactive Authorization Request containing an OpenID4VP Authorization Response:

```http
POST /iae HTTP/1.1
Host: server.example.com
OAuth-Client-Attestation: eyJ...
OAuth-Client-Attestation-PoP: eyJ...
Content-Type: application/x-www-form-urlencoded

auth_session=wxroVrBY2MCq4dDNGXACS
&openid4vp_response=...
```

The following is a non-normative example of the `openid4vp_response` JSON object:

```json
{
  "vp_token": ...
}

```

The following is a non-normative example of the `openid4vp_response` JSON object with an encrypted response:

```json
{
  "response": ...
}

```

The following is a non-normative example of the `openid4vp_response` JSON object with an Authorization Error Response.

```json
{
  "error": "invalid_request",
  "error_description": "unsupported client_id_prefix"
}
```

Note: This mechanism can only be used for interactions with the same Wallet that started the issuance process.

#### Redirect to Web {#iae-redirect-to-web}

If the type is `redirect_to_web`, the Authorization Server is indicating that the authorization process must continue via interactions with the user in a web browser.

In this case, the Authorization server MUST include the key `request_uri` in the response.
The Wallet MUST use the `request_uri` value to build an Authorization Request as defined in Section 4 of [@!RFC9126] and complete the rest of the authorization process as defined there.
The Wallet MUST only use a `request_uri` value once.
Authorization servers SHOULD treat `request_uri` values as one-time use but MAY allow for duplicate requests due to a user reloading/refreshing their user agent. An expired request_uri MUST be rejected as invalid.
The Authorization Server MAY include the `expires_in` key as defined in [@!RFC9126].

Non-normative Example:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "status": "require_interaction",
  "type": "redirect_to_web",
  "request_uri": "urn:ietf:params:oauth:request_uri:6esc_11ACC5bwc014ltc14eY22c",
  "expires_in": 60
}
```

Once this phase of the Authorization process is completed, the Authorization Server MUST redirect back to the Wallet as per [@RFC6749]. If the Authorization process is complete when this redirect occurs, the Authorization Server returns a response with the `code` parameter as per Section 1.3.1 of [@RFC6749]. If the Authorization process is not complete when this redirect occurs, the Authorization Server returns a response with the `auth_session` parameter. In the event a Wallet receives a response from the Authorization Server which features the `auth_session` parameter, the Wallet MUST make a follow-up request as per (#follow-up-request) to continue the Authorization process. In the event that PKCE as defined in [@RFC7636] was used in the initial authorization request to the interactive authorization endpoint, the Authorization Server MUST enforce the correct usage of the `code_verifier` in the follow-up request that follows the completion of the `redirect_to_web` interaction.

To ensure the security of the `redirect_to_web` flow, the redirect URI MUST be an `https` URL as per Section 7.2 of [@!RFC8252]. The Wallet MUST NOT use an embedded user-agent to perform the `redirect_to_web` flow. The considerations in Section 8.12 of [@!RFC8252] apply. Platform-specific implementation details are provided in Appendix B of the same document.

A non-normative example of a follow-up request featuring PKCE:

```
POST /iae HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

auth_session=wxroVrBY2MCq4dDNGXACS&code_verifier=avjebhrnqwketh
```

#### Custom Interaction Extensions {#iae-custom-extensions}

Additional, custom types of interactions MAY be defined by extensions of this specification to enable other types of interactions, for example, by interacting with a smart card.
It is RECOMMENDED to use this extension point instead of modifying the OAuth protocol in order to facilitate interactions that require interactions with native components of the Wallet application.
See (#iae-security) for additional security considerations.

In the following non-normative example, this extension point is used to read the Betelgeuse Intergalactic ID card through an NFC interface in the Wallet. A token called `biic_token` is used to start the process.

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "status": "require_interaction",
  "type": "betelgeuse_intergalactic_id_card",
  "biic_token": "73475cb40a568e8da8a045ced110137e159f890ac4da883b6b17dc651b3a8049"
}
```

#### Preventing Session Fixation Attacks {#iae-security}

Authorization Servers MUST ensure that the user interaction (OpenID4VP presentation, redirect to web, or a custom interaction) is securely bound to the authorization process in order avoid Session Fixation Attacks as described in Section 14.2 of [@!OpenID4VP].
This can be achieved by securely linking all requests following the initial Interactive Authorization Request.
For OpenID4VP presentations, the Authorization Server MUST associate the `nonce` value used in the Presentation with the `auth_session` value and verify that the Presentation delivered from the Wallet to the Verifier uses the same nonce.

Custom extensions ((#iae-custom-extensions)) MUST ensure an equivalent binding.
Authorization Servers can usually achieve this by providing a nonce for use in the custom process (`biic_token` in the example above) and either only responding to the Interactive Authorization Request Endpoint (as done by (#iae-require-presentation)) or returning a non-predictable value from the process to the Interactive Authorization Request Endpoint that can be verified.

#### Preventing Forwarding of Interactive Authorization Endpoint Presentation Requests

In ecosystems with multiple Authorization Servers that may potentially use the Interactive Authorization Endpoint, there is a risk that a malicious (or compromised) Authorization Server forwards an Interactive Authorization Response containing a Interaction Required Response that it itself has acquired from another Authorization Server.
This may lead to the malicious Authorization Server gaining access to Credentials issued by the other Authorization Server without the End-User's consent.

Custom extensions ((#iae-custom-extensions)) MUST ensure that this attack is prevented by ensuring one or both of the following:

 1. The Wallet is able to detect that a request is not presented by the party that initiated the Interactive Authorization Request. In the case of the (#iae-require-presentation) interaction with a signed Presentation request, this is achieved by the Wallet verifying the `expected_origins` parameter in the request, which contains the derived Origin of the Interactive Authorization Endpoint that initiated the request.
 2. The Authorization Server is able to detect that the request was forwarded to a different endpoint. In the case of the (#iae-require-presentation) interaction, this is achieved for both signed and unsigned requests by the binding the Interactive Authorization Endpoint to the Verifiable Presentation (see "Interactive Authorization Endpoint Binding" sections under (#format-profiles)), which is then verified by the Authorization Server.

### Authorization Code Response {#iae-authorization-code-response}

Once the Authorization Server has successfully processed the Interactive Authorization Request, it MUST respond with a 200 OK response using the `application/json` media type containing a `code` parameter, carrying the Authorization Code as defined in [@!RFC6749].
The `status` key MUST be set to `ok` in this case.

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "code": "uY29tL2F1dGhlbnRpY",
  "status": "ok"
}
```

The Wallet MUST use this authorization code in the subsequent Token Request to the Token Endpoint.

### Interactive Authorization Error Response {#iae-error-response}

In addition to the error processing rules defined in Section 2.3 of [@RFC9126], this specification defines the following error codes for the Interactive Authorization Endpoint:

* `missing_interaction_type`: The `interaction_types_supported` parameter in the Interactive Authorization Request does not include all interaction types required to complete all phases of the authorization process.

The following is an example of an error response from the Interactive Authorization Endpoint:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-cache, no-store

{
  "error": "missing_interaction_type",
  "error_description":
    "interaction_types_supported in the request is missing the required interaction type 'openid4vp_presentation'"
}
```

# Token Endpoint {#token-endpoint}

The Token Endpoint issues an Access Token and, optionally, a Refresh Token in exchange for the Authorization Code that Client obtained in a successful Authorization Response. It is used in the same manner as defined in [@!RFC6749]. Implementers SHOULD follow the best current practices for OAuth 2.0 Security given in [@!BCP240], see (#securitybcp).

## Token Request {#token-request}

The Token Request is made as defined in Section 4.1.3 of [@!RFC6749].

The following are the extension parameters to the Token Request used in the Pre-Authorized Code Flow defined in (#pre-authz-code-flow):

* `pre-authorized_code`: The code representing the authorization to obtain Credentials of a certain type. This parameter MUST be present if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.
* `tx_code`: OPTIONAL. String value containing a Transaction Code value itself. This value MUST be present if a `tx_code` object was present in the Credential Offer (including if the object was empty). This parameter MUST only be used if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.

Requirements around how the Wallet identifies and, if applicable, authenticates itself with the Authorization Server in the Token Request depend on the Client type defined in Section 2.1 of [@!RFC6749] and the Client authentication method indicated in the `token_endpoint_auth_method` Client metadata (as defined in [@!RFC7591]). The requirements specified in Sections 4.1.3 and 3.2.1 of [@!RFC6749] MUST be followed.

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
&redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSU...
```

### Request Credential Issuance using `authorization_details` Parameter

Credential Issuers MAY support requesting authorization to issue a Credential using the `authorization_details` parameter. This is particularly useful if the Credential Issuer offered multiple Credential Configurations in the Credential Offer of a Pre-Authorized Code Flow. 

The Wallet can use `authorization_details` in the Token Request to request a specific Credential Configuration in both the Authorization Code Flow and the Pre-Authorized Code Flow. The value of the `authorization_details` parameter is defined in (#authorization-details).

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

The Wallet may send the `authorization_details` parameter in the Token Request even when the parameter has been previously sent in the [Authorization Request](#credential-authz-request) as described in Section 6.1 of [@!RFC9396]. It allows the AS to make a decision based on whether the Wallet is asking for more or less access than the previous request. With respect to `authorization_details` items using the `credential_configuration_id` introduced in this specification, it is RECOMMENDED that the AS would accept a request from the Wallet containing a subset of `credential_configuration_id` parameters received in the original [Authorization Request](#credential-authz-request) and issue a token for the reduced set.

## Successful Token Response {#token-response}

Token Responses are made as defined in [@!RFC6749].

The Authorization Server might decide to authorize issuance of multiple instances for each Credential requested in the Authorization Request. Each Credential instance is described using the same entry in the `credential_configurations_supported` Credential Issuer metadata, but contains different claim values or different subset of claims within the claims set identified by the `credential_configuration_id`.

In addition to the response parameters defined in [@!RFC6749], the Authorization Server MAY return the following parameters:

* `authorization_details`: REQUIRED when the `authorization_details` parameter, as defined in (#authorization-details), is used in either the [Authorization Request](#credential-authz-request) or [Token Request](#token-request). OPTIONAL when `scope` parameter was used to request issuance of a Credential of a certain Credential Configuration. It is a non-empty array of objects, as defined in Section 7 of [@!RFC9396]. In addition to the parameters defined in (#authorization-details), this specification defines the following parameter to be used with the authorization details type `openid_credential` in the Token Response:
  * `credential_identifiers`: REQUIRED. A non-empty array of strings, each uniquely identifying a Credential Dataset that can be issued using the Access Token returned in this response. Each of these Credential Datasets corresponds to the Credential Configuration referenced in the `credential_configuration_id` parameter. The Wallet MUST use these identifiers together with an Access Token in subsequent Credential Requests. See (#identifying_credential) for the summary of the options how requested Credential(s) are identified throughout the Issuance flow.


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
  "token_type": "Bearer",
  "expires_in": 86400,
  "authorization_details": [
    {
      "type": "openid_credential",
      "credential_configuration_id": "UniversityDegreeCredential",
      "credential_identifiers": [
        "CivilEngineeringDegree-2023",
        "ElectricalEngineeringDegree-2023"
      ]
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

Below is a non-normative example of a Token Error Response:

```
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

A request for a nonce is made by sending an HTTP POST request to the URL provided in the `nonce_endpoint` Credential Issuer Metadata parameter. The Nonce Endpoint is not a protected resource, meaning the Wallet does not need to supply an access token to access it.

Below is a non-normative example of a Nonce Request:

```
POST /nonce HTTP/1.1
Host: credential-issuer.example.com
Content-Length: 0
```

## Nonce Response {#nonce-response}

The Credential Issuer provides a nonce value in an HTTP response with a 2xx status code and the following parameters included as top-level members in the message body of the HTTP response using the application/json media type:

* `c_nonce`: REQUIRED. String containing a challenge to be used when creating a proof of possession of the key (see (#credential-request)). It is at the discretion of the Credential Issuer when to return a new challenge value as opposed to the one returned in the previous request. New challenge values MUST be unpredictable.

Due to the temporal nature of the `c_nonce` value, the Credential Issuer MUST make the response uncacheable by adding a `Cache-Control` header field including the value `no-store`.

 The Credential Issuer MAY provide a DPoP nonce in an HTTP header as defined in Section 8.2 of [@!RFC9449]. In this case, the Wallet uses the new nonce value in the DPoP proof when presenting an access token at the Credential Endpoint.

Below is a non-normative example of a Nonce Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
DPoP-Nonce: eyJ7S_zG.eyJH0-Z.HX4w-7v

{
  "c_nonce": "wKI4LT17ac15ES9bw8ac4"
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

The issued Credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the Credential. Cryptographic Key Binding allows the Verifier to verify during the presentation of a Credential that the End-User presenting a Credential is the same End-User to whom that Credential was issued. For non-cryptographic types of binding and Credentials issued without any binding, see the Implementation Considerations in (#claims-based-binding) and (#no-binding).

Note: Claims in the Credential are about the subject of the Credential, which is often the End-User who possesses it.

For Cryptographic Key Binding, the Client has different options to provide Cryptographic Key Binding material for a requested Credential within a proof of a certain proof type. A proof type may provide the cryptographic public key(s) either with corresponding proof(s) of possession of the private key(s) or with key attestation(s). Proof types are defined in (#proof-types).

## Credential Request {#credential-request}

A Client makes a Credential Request to the Credential Endpoint by sending the following parameters in the entity-body of an HTTP POST request. The Credential Request MAY be encrypted (on top of TLS) using the `credential_request_encryption` parameter in (#credential-issuer-metadata) as specified in (#encrypted-messages).

* `credential_identifier`: REQUIRED when an Authorization Details of type `openid_credential` was returned from the Token Response. It MUST NOT be used otherwise. A string that identifies a Credential Dataset that is requested for issuance. When this parameter is used, the `credential_configuration_id` MUST NOT be present.
* `credential_configuration_id`: REQUIRED if a `credential_identifiers` parameter was not returned from the Token Response as part of the `authorization_details` parameter. It MUST NOT be used otherwise. String that uniquely identifies one of the keys in the name/value pairs stored in the `credential_configurations_supported` Credential Issuer metadata. The corresponding object in the `credential_configurations_supported` map MUST contain one of the value(s) used in the `scope` parameter in the Authorization Request. When this parameter is used, the `credential_identifier` MUST NOT be present.
* `proofs`: OPTIONAL. Object providing one or more proof of possessions of the cryptographic key material to which the issued Credential instances will be bound to. The `proofs` parameter contains exactly one parameter named as the proof type in (#proof-types), the value set for this parameter is a non-empty array containing parameters as defined by the corresponding proof type.
* `credential_response_encryption`: OPTIONAL. Object containing information for encrypting the Credential Response. If this request element is not present, the corresponding credential response returned is not encrypted.
    * `jwk`: REQUIRED. Object containing a single public key as a JWK used for encrypting the Credential Response.
    * `enc`: REQUIRED. JWE [@!RFC7516] `enc` algorithm [@!RFC7518] for encrypting Credential Responses.
    * `zip`: OPTIONAL. JWE [@!RFC7516] `zip` algorithm [@!RFC7518] for compressing Credential Responses prior to encryption. If absent then compression MUST not be used.

See (#identifying_credential) for the summary of the options how requested Credential(s) are identified throughout the Issuance flow.

The proof type contained in the `proofs` parameter is an extension point that enables the use of different types of proofs for different cryptographic schemes.

The proof(s) in the `proofs` parameter MUST incorporate the Credential Issuer Identifier (audience) and, if the Credential Issuer has a Nonce Endpoint, a `c_nonce` value to allow the Credential Issuer to detect freshness. The way that data is incorporated depends on the key proof type. In a JWT, for example, the `c_nonce` value is conveyed in the `nonce` claim, whereas the audience is conveyed in the `aud` claim. In a Linked Data proof, for example, the `c_nonce` is included as the `challenge` element in the key proof object and the Credential Issuer (the intended audience) is included as the `domain` element.

The `proofs` parameter MUST be present if the `proof_types_supported` parameter is present in the `credential_configurations_supported` parameter of the Issuer metadata for the requested Credential.

The `c_nonce` value is retrieved from the Nonce Endpoint as defined in (#nonce-endpoint).

Additional Credential Request parameters MAY be defined and used.
The Credential Issuer MUST ignore any unrecognized parameters.

The Credential Issuer indicates support for encrypted requests by including the `credential_request_encryption` parameter in the Credential Issuer Metadata. The Client MAY encrypt the request when `encryption_required` is `false` and MUST do so when `encryption_required` is `true`. 

When performing Credential Request encryption, the Client MUST encode the information in the Credential Request in a JWT as specified by (#encrypted-messages), using the parameters from the `credential_request_encryption` object in the Credential Issuer Metadata.

If the Credential Request is not encrypted, the media type of the request MUST be set to `application/json`.

Below is a non-normative example of a Credential Request for a Credential in [@ISO.18013-5] format using the Credential configuration identifier and a key proof type `jwt` (with line breaks within values for display purposes only):

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "credential_configuration_id": "org.iso.18013.5.1.mDL",
  "proofs": {
    "jwt": [
      "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8x
       IiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ"
    ]
  }
}
```

Below is a non-normative example of a Credential Request for two Credential instances in an IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format using a Credential identifier from the Token Response and key proof type `jwt` (with line breaks within values for display purposes only):

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "credential_identifier": "CivilEngineeringDegree-2023",
  "proofs": {
    "jwt": [
      "eyJ0eXAiOiJvcGVuaWQ0dmNpLXByb29mK2p3dCIsImFsZyI6IkVTMjU2IiwiandrIjp7Imt0
       eSI6IkVDIiwiY3J2IjoiUC0yNTYiLCJ4IjoiblVXQW9BdjNYWml0aDhFN2kxOU9kYXhPTFlG
       T3dNLVoyRXVNMDJUaXJUNCIsInkiOiJIc2tIVThCalVpMVU5WHFpN1N3bWo4Z3dBS18weGtj
       RGpFV183MVNvc0VZIn19",
      "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8x
       IiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ"
    ]
  }
}
```

Below is a non-normative example of a Credential Request for one Credential in W3C VCDM format using a Credential identifier from the Token Response and key proof type `di_vp` (with line breaks within values for display purposes only):

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "credential_identifier": "CivilEngineeringDegree-2023",
  "proofs": {
    "di_vp": [
      {
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
            "verificationMethod": "did:key:z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mq
             h5XPvLsrPsro#z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mqh5XPvLsrPsro",
            "created": "2023-03-01T14:56:29.280619Z",
            "challenge": "82d4cb36-11f6-4273-b9c6-df1ac0ff17e9",
            "domain": "did:web:audience.company.com",
            "proofValue": "z5hrbHzZiqXHNpLq6i7zePEUcUzEbZKmWfNQzXcUXUrqF7bykQ7A
             CiWFyZdT2HcptF1zd1t7NhfQSdqrbPEjZceg7"
          }
        ]
      }
    ]
  }
}
```

The Credential Issuer indicates support for encrypted responses by including the `credential_response_encryption` parameter in the Credential Issuer Metadata. The Client MAY request encrypted responses by providing its encryption parameters in the Credential Request when `encryption_required` is `false` and MUST do so when `encryption_required` is `true`. Credential Request encryption MUST be used if the `credential_response_encryption` parameter is included, to prevent it being substituted by an attacker.

## Credential Response {#credential-response}

The Credential Response can either be returned immediately or in a deferred manner. The response can contain one or more Credentials with the same Credential Configuration and Credential Dataset depending on the Credential Request:

* If the Credential Issuer is able to immediately issue the requested Credentials, it MUST respond with the HTTP status code 200 (see Section 15.3.3 of [@!RFC9110]).
* If the Credential Issuer is not able to immediately issue the requested credentials (e.g., due to a manual review process being required or the data used to issue the credential is not ready yet), the Credential Issuer MUST return a response with a `transaction_id` parameter. In this case, the Credential Issuer MUST also use the HTTP status code 202 for the response. The `transaction_id` MAY be used by the Client at a later time at the Deferred Credential endpoint.

If the Client requested an encrypted response by including the `credential_response_encryption` object in the request, the Credential Issuer MUST encode the information in the Credential Response as specified by (#encrypted-messages), using the parameters from the `credential_response_encryption` object. Note that this is done regardless of the content. 

If the Credential Response is not encrypted, the media type of the response MUST be set to `application/json`.

The following parameters are used in the JSON-encoded Credential Response body:

* `credentials`: OPTIONAL. Contains an array of one or more issued Credentials. It MUST NOT be used if the `transaction_id` parameter is present. The elements of the array MUST be objects. The number of elements in the `credentials` array matches the number of keys that the Wallet has provided via the `proofs` parameter of the Credential Request, unless the Issuer decides to issue fewer Credentials. Each key provided by the Wallet is used to bind to, at most, one Credential. This specification defines the following parameters to be used inside this object:
   * `credential`: REQUIRED. Contains one issued Credential. The encoding of the Credential depends on the Credential Format and MAY be a string or an object. Credential Formats expressed as binary data MUST be base64url-encoded and returned as a string. More details are defined in the Credential Format Profiles in (#format-profiles).
* `transaction_id`: OPTIONAL. String identifying a Deferred Issuance transaction. This parameter is contained in the response if the Credential Issuer cannot immediately issue the Credential. The value is subsequently used to obtain the respective Credential with the Deferred Credential Endpoint (see (#deferred-credential-issuance)). It MUST not be used if the `credentials` parameter is present. It MUST be invalidated after the Credential for which it was meant has been obtained by the Wallet.
* `interval`: REQUIRED if `transaction_id` is present. Contains a positive number that represents the minimum amount of time in seconds that the Wallet SHOULD wait after receiving the response before sending a new request to the Deferred Credential Endpoint. It MUST NOT be used if the `credentials` parameter is present.
* `notification_id`: OPTIONAL. String identifying one or more Credentials issued in one Credential Response. It MUST be included in the Notification Request as defined in (#notification). It MUST not be used if the `credentials` parameter is not present.

Additional Credential Response parameters MAY be defined and used. The Wallet MUST ignore any unrecognized parameters.

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
  "transaction_id": "8xLOxBtZp8",
  "interval" : 3600
}
```

### Credential Error Response {#credential-error-response}

When the Credential Request is invalid or unauthorized, the Credential Issuer constructs the error response as defined in this section.

#### Authorization Errors {#authorization-errors}

If the Credential Request does not contain an Access Token that enables issuance of a requested Credential, the Credential Endpoint returns an authorization error response such as defined in Section 3 of [@!RFC6750].

#### Credential Request Errors {#credential-request-errors}

For errors related to the Credential Request's payload, such as issues with `type`, `format`, `proofs`, encryption parameters, or if the request is denied, the specific error codes from this section MUST be used instead of the generic `invalid_request` parameter defined in Section 3.1 of [@!RFC6750].

If the Wallet is requesting the issuance of a Credential that is not supported by the Credential Endpoint, the HTTP response MUST use the HTTP status code 400 (Bad Request) and set the content type to `application/json` with the following parameters in the JSON-encoded response body:

* `error`: REQUIRED. The `error` parameter SHOULD be a single ASCII [@!USASCII] error code from the following:
  * `invalid_credential_request`: The Credential Request is missing a required parameter, includes an unsupported parameter or parameter value, repeats the same parameter, or is otherwise malformed.
  * `unknown_credential_configuration`: Requested Credential Configuration is unknown.
  * `unknown_credential_identifier`: Requested Credential identifier is unknown.
  * `invalid_proof`: The `proofs` parameter in the Credential Request is invalid: (1) if the field is missing, or (2) one of the provided key proofs is invalid, or (3) if at least one of the key proofs does not contain a `c_nonce` value (refer to (#nonce-response)).
  * `invalid_nonce`: The `proofs` parameter in the Credential Request uses an invalid nonce: at least one of the key proofs contains an invalid `c_nonce` value. The wallet should retrieve a new `c_nonce` value (refer to (#nonce-endpoint)).
  * `invalid_encryption_parameters`: This error occurs when the encryption parameters in the Credential Request are either invalid or missing. In the latter case, it indicates that the Credential Issuer requires the Credential Response to be sent encrypted, but the Credential Request does not contain the necessary encryption parameters.
  * `credential_request_denied`: The Credential Request has not been accepted by the Credential Issuer. The Wallet SHOULD treat this error as unrecoverable, meaning if received from a Credential Issuer the Credential cannot be issued. 
* `error_description`: OPTIONAL. The `error_description` parameter MUST be a human-readable ASCII [@!USASCII] text, providing any additional information used to assist the Client implementers in understanding the occurred error. The values for the `error_description` parameter MUST NOT include characters outside the set `%x20-21 / %x23-5B / %x5D-7E`.

The usage of these parameters takes precedence over the `invalid_request` parameter defined in (#authorization-errors), since they provide more details about the errors.

Note that Credential Error Responses are never encrypted, even if a valid Credential Response would have been.

The following is a non-normative example of a Credential Error Response where an unsupported Credential Format was requested:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "unknown_credential_configuration"
}
```

# Deferred Credential Endpoint {#deferred-credential-issuance}

This endpoint is used to issue one or more Credentials previously requested at the Credential Endpoint in cases where the Credential Issuer was not able to immediately issue this Credential. Support for this endpoint is OPTIONAL.

The Wallet MUST present to the Deferred Endpoint an Access Token that is valid for the issuance of the Credential(s) previously requested at the Credential Endpoint. 

Communication with the Deferred Credential Endpoint MUST utilize TLS. 

## Deferred Credential Request {#deferred-credential-request}

The Deferred Credential Request is an HTTP POST request. The Deferred Credential Request MAY be encrypted (on top of TLS) using the `credential_request_encryption` parameter in (#credential-issuer-metadata) as specified in (#encrypted-messages).

The following parameters are used in the Deferred Credential Request:

* `transaction_id`: REQUIRED. String identifying a Deferred Issuance transaction.
* `credential_response_encryption`: OPTIONAL. as defined in (#credential-request).

The Credential Issuer MUST invalidate the `transaction_id` after the Credential for which it was meant has been obtained by the Wallet.

Additional Deferred Credential Request parameters MAY be defined and used.
The Credential Issuer MUST ignore any unrecognized parameters.

The Credential Issuer indicates support for encrypted requests by including the `credential_request_encryption` parameter in the Credential Issuer Metadata. The Client MAY encrypt the request when `encryption_required` is `false` and MUST do so when `encryption_required` is `true`.

When performing Deferred Credential Request encryption, the Client MUST encode the information in the Deferred Credential Request in a JWT as specified by (#encrypted-messages), using the parameters from the `credential_request_encryption` object in the Credential Issuer Metadata. 

If the Deferred Credential Request is not encrypted, the media type of the request MUST be set to `application/json`.

The following is a non-normative example of a Deferred Credential Request:

```
POST /deferred_credential HTTP/1.1 
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "transaction_id": "8xLOxBtZp8"
}
```

The Credential Issuer indicates support for encrypted responses by including the `credential_response_encryption` parameter in the Credential Issuer Metadata. The Client MAY request encrypted responses by providing its encryption parameters in the Deferred Credential Request when `encryption_required` is `false` and MUST do so when `encryption_required` is `true`. Note that this object will be used for encrypting the response, regardless of what was sent in the initial Credential Request. If it is not included encryption will not be performed. Deferred Credential Request encryption MUST but used if the `credential_response_encryption` parameter is included, to prevent it being substituted by an attacker.

## Deferred Credential Response {#deferred-credential-response}

A Deferred Credential Response may either contain the requested Credentials or further defer the issuance:

* If the Credential Issuer is able to issue the requested Credentials, the Deferred Credential Response MUST use the `credentials` parameter as defined in (#credential-response) and MUST respond with the HTTP status code 200 (see Section 15.3.3 of [@!RFC9110]).
* If the Credential Issuer still requires more time, the Deferred Credential Response MUST use the `interval` and `transaction_id` parameters as defined in (#credential-response) and it MUST respond with the HTTP status code 202 (see Section 15.3.3 of [@!RFC9110]). The value of `transaction_id` MUST be same as the value of `transaction_id` in the Deferred Credential Request.

The Deferred Credential Response MAY use the `notification_id` parameter as defined in (#credential-response).

Additional Deferred Credential Response parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

If the Client requested an encrypted response by including the `credential_response_encryption` object in the request, the Credential Issuer MUST encode the information in the Deferred Credential Response as specified by (#encrypted-messages), using the parameters from the `credential_response_encryption` object. Note that this is done regardless of the content. The `credential_response_encryption` object may be different from the one included in the initial Credential Request so the Credential Issuer MUST use the newly provided one. This is to simplify key management in the case of longer deferred issuance. 

If the Deferred Credential Response is not encrypted, the media type of the response MUST be set to `application/json`.

The following is a non-normative example of a Deferred Credential Response containing Credentials:

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

The following is a non-normative example of a Deferred Credential Response, where the Credential Issuer still requires more time:

```
HTTP/1.1 202 OK
Content-Type: application/json

{
  "transaction_id": "8xLOxBtZp8",
  "interval": 86400
}
```

## Deferred Credential Error Response {#deferred-credential-error-response}

When the Deferred Credential Request is invalid, the Credential Issuer constructs the error response as defined in (#credential-error-response).

The following additional error code is specified in addition to those already defined in (#credential-request-errors):

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

In the event the Credential Issuer can no longer issue the credential(s), the `credential_request_denied` error code as defined in (#credential-request-errors) should be used in response to a request. A wallet upon receiving this error SHOULD stop making requests to the deferred credential endpoint for the given `transaction_id`. 

# Encrypted Credential Requests and Responses {#encrypted-messages}
Encryption of requests and responses for the Credential and Deferred Credential Endpoints is performed as follows:

The contents of the message MUST be encoded as a JWT as described in [@!RFC7519]. The media type MUST be set to `application/jwt`.

The Public Key used to encrypt the message is selected based on the context. In the case where multiple public keys are available, any may be selected based on the information about each key, such as the `kty` (Key Type), `use` (Public Key Use), `alg` (Algorithm), and other JWK parameters. The `alg` parameter MUST be present. The JWE `alg` algorithm used MUST be equal to the `alg` value of the chosen JWK. If the selected public key contains a `kid` parameter, the JWE MUST include the same value in the `kid` JWE Header Parameter (as defined in [@!RFC7516, Section 4.1.6]) of the encrypted message. This enables the easy identification of the specific public key that was used to encrypt the message. The JWE `enc` content encryption algorithm used is obtained based on context.

If a `zip` (Compression Algorithm) value is specified, then compression is performed before encryption, as specified in [@!RFC7516]. If absent, no compression is performed.

When encryption of a message was required but the received message is unencrypted, it SHOULD be rejected.

For security considerations see (#encryption-security-considersations)

# Notification Endpoint {#notification-endpoint}

This endpoint is used by the Wallet to notify the Credential Issuer of certain events for issued Credentials. These events enable the Credential Issuer to take subsequent actions after issuance. The Credential Issuer needs to return one `notification_id` parameter per Credential Response or Deferred Credential Response for the Wallet to be able to use this endpoint. Support for this endpoint is OPTIONAL. The Issuer cannot assume that a notification will be sent for every issued Credential since the use of this Endpoint is not mandatory for the Wallet.

The Wallet MUST present to the Notification Endpoint a valid Access Token issued at the Token Endpoint as defined in (#token-endpoint). 

A Credential Issuer that requires a request to the Notification Endpoint MUST ensure the Access Token issued by the Authorization Server is valid at the Notification Endpoint.

The notification from the Wallet is idempotent. When the Credential Issuer receives multiple identical calls from the Wallet for the same `notification_id`, it returns success. Due to the network errors, there are no guarantees that a Credential Issuer will receive a notification within a certain time period or at all.

Communication with the Notification Endpoint MUST utilize TLS.

## Notification Request {#notification}

The Wallet sends an HTTP POST request to the Notification Endpoint with the following parameters in the entity-body and using the `application/json` media type. If the Wallet supports the Notification Endpoint, the Wallet MAY send one or more Notification Requests per `notification_id` value received.

* `notification_id`: REQUIRED. String received in the Credential Response or Deferred Credential Response identifying an issuance flow that contained one or more Credentials with the same Credential Configuration and Credential Dataset.
* `event`: REQUIRED. Type of the notification event. It MUST be a case sensitive string whose value is either `credential_accepted`, `credential_failure`, or `credential_deleted`. `credential_accepted` is to be used when the Credentials were successfully stored in the Wallet, with or without user action. `credential_deleted` is to be used when the unsuccessful Credential issuance was caused by a user action. In all other unsuccessful cases, `credential_failure` is to be used. Partial errors during issuance of multiple Credentials as a batch (e.g., one of the Credentials could not be stored) MUST be treated as the overall issuance flow failing.
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

Below is a non-normative example of a Notification Request when a Credential was rejected by the End-User:

```
POST /notification HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: Bearer czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
  "notification_id": "3fwe98js",
  "event": "credential_deleted",
  "event_description": "User rejected the issued Credential."
}
```

Below is a non-normative example of a Notification Request when a Credential could not be stored in the Wallet:

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

Below is a non-normative example of a response to a successful Notification Request:

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

Credential Issuers publishing metadata MUST make a JSON document available at the path formed by inserting the string `/.well-known/openid-credential-issuer` into the Credential Issuer Identifier between the host component and the path component, if any.

For example, the metadata for the Credential Issuer Identifier `https://issuer.example.com/tenant` would be retrieved from `https://issuer.example.com/.well-known/openid-credential-issuer/tenant`. The metadata for the Credential Issuer Identifier `https://tenant.issuer.example.com` would be retrieved from `https://tenant.issuer.example.com/.well-known/openid-credential-issuer`.

Communication with the Credential Issuer Metadata Endpoint MUST utilize TLS.

To fetch the Credential Issuer Metadata, the Wallet MUST send an HTTP request using the GET method and the path formed following the steps above. The Wallet is RECOMMENDED to send an `Accept` header in the HTTP GET request to indicate the Content Type(s) it supports, and by doing so, signaling whether it supports signed metadata.

The Credential Issuer MUST respond with HTTP Status Code 200 and return the Credential Issuer Metadata containing the parameters defined in (#credential-issuer-parameters) as either

* an unsigned JSON document using the media type `application/json`, or
* a signed JSON Web Token (JWT) containing the Credential Issuer Metadata in its payload using the media type `application/jwt`.

The Credential Issuer MUST support returning metadata in an unsigned form 'application/json' and MAY support returning it in a signed form 'application/jwt'. In all cases the Credential Issuer MUST indicate the media type of the returned Metadata using the HTTP `Content-Type` header. It is RECOMMENDED for Credential Issuers to respond with a `Content-Type` matching to the Wallet's requested `Accept` header when the requested content type is supported.

The Wallet is RECOMMENDED to send an `Accept-Language` header in the HTTP GET request to indicate the language(s) preferred for display. It is up to the Credential Issuer whether to:

* send a subset the metadata containing internationalized display data for one or all of the requested languages and indicate returned languages using the HTTP `Content-Language` Header, or
* ignore the `Accept-Language` Header and send all supported languages or any chosen subset.

The language(s) in HTTP `Accept-Language` and `Content-Language` Headers MUST use the values defined in [@!RFC3066]. 

Below is a non-normative example of a Credential Issuer Metadata request:

```
GET /.well-known/openid-credential-issuer HTTP/1.1
Host: server.example.com
Accept-Language: fr-ch, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5
```

### Signed Metadata {#credential-issuer-signed-metadata}

The signed metadata MUST be secured using a JSON Web Signature (JWS) [@!RFC7515] and contain the following elements:

* in the JOSE header,
  * `alg`: REQUIRED. A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry [@IANA.JOSE]. It MUST NOT be `none` or an identifier for a symmetric algorithm (MAC).
  * `typ`: REQUIRED. MUST be `openidvci-issuer-metadata+jwt`, which explicitly types the key proof JWT as recommended in Section 3.11 of [@!RFC8725].

* in the JWS payload,
  * `iss`: OPTIONAL. String denoting the party attesting to the claims in the signed metadata
  * `sub`: REQUIRED. String matching the Credential Issuer Identifier
  * `iat`: REQUIRED. Integer for the time at which the Credential Issuer Metadata was issued using the syntax defined in [@!RFC7519].
  * `exp`: OPTIONAL. Integer for the time at which the Credential Issuer Metadata is expiring, using the syntax defined in [@!RFC7519].

All [metadata parameters](#credential-issuer-parameters) used by the Credential Issuer MUST be added as top-level claims in the JWS payload.

When requesting signed metadata, the Wallet MUST establish trust in the signer of the metadata. Otherwise, the Wallet MUST reject the signed metadata. When validating the signature, the Wallet obtains the keys to validate the signature before processing the metadata, e.g., using JOSE header parameters like `x5c`, `kid` or `trust_chain` to convey the public key. The concrete mechanisms for doing this are out of scope of this specification.

See (#additional-issuer-metadata-examples) for a non-normative example of signed Credential Issuer Metadata.

### Credential Issuer Metadata Parameters {#credential-issuer-parameters}

This specification defines the following Credential Issuer Metadata parameters:

* `credential_issuer`: REQUIRED. The Credential Issuer's identifier, as defined in (#credential-issuer-identifier). The value MUST be identical to the Credential Issuer's identifier value into which the well-known URI string was inserted to create the URL used to retrieve the metadata. If these values are not identical (when compared using a simple string comparison with no normalization), the data contained in the response MUST NOT be used.
* `authorization_servers`: OPTIONAL. A non-empty array of strings, where each string is an identifier of the OAuth 2.0 Authorization Server (as defined in [@!RFC8414]) the Credential Issuer relies on for authorization. If this parameter is omitted, the entity providing the Credential Issuer is also acting as the Authorization Server, i.e., the Credential Issuer's identifier is used to obtain the Authorization Server metadata. The actual OAuth 2.0 Authorization Server metadata is obtained from the `oauth-authorization-server` well-known location as defined in Section 3 of [@!RFC8414]. When there are multiple entries in the array, the Wallet may be able to determine which Authorization Server to use by querying the metadata; for example, by examining the `grant_types_supported` values, the Wallet can filter the server to use based on the grant type it plans to use. When the Wallet is using `authorization_server` parameter in the Credential Offer as a hint to determine which Authorization Server to use out of multiple, the Wallet MUST NOT proceed with the flow if the `authorization_server` Credential Offer parameter value does not match any of the entries in the `authorization_servers` array.
* `credential_endpoint`: REQUIRED. URL of the Credential Issuer's Credential Endpoint, as defined in (#credential-request). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components.
* `nonce_endpoint`: OPTIONAL. URL of the Credential Issuer's Nonce Endpoint, as defined in (#nonce-endpoint). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not require the use of `c_nonce`.
* `deferred_credential_endpoint`: OPTIONAL. URL of the Credential Issuer's Deferred Credential Endpoint, as defined in (#deferred-credential-issuance). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Deferred Credential Endpoint.
* `notification_endpoint`: OPTIONAL. URL of the Credential Issuer's Notification Endpoint, as defined in (#notification-endpoint). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Notification Endpoint.
* `credential_request_encryption`: OPTIONAL. Object containing information about whether the Credential Issuer supports encryption of the Credential Request on top of TLS.
  * `jwks`: REQUIRED. A JSON Web Key Set, as defined in [@!RFC7591], that contains one or more public keys, to be used by the Wallet as an input to a key agreement for encryption of the Credential Request. Each JWK in the set MUST have a kid (Key ID) parameter that uniquely identifies the key.
  * `enc_values_supported`: REQUIRED. A non-empty array containing a list of the JWE [@!RFC7516] encryption algorithms (`enc` values) [@!RFC7518] supported by the Credential Endpoint to decode the Credential Request from a JWT [@!RFC7519].
  * `zip_values_supported`: OPTIONAL. A non-empty array containing a list of the JWE [@!RFC7516] compression algorithms (`zip` values) [@!RFC7518] supported by the Credential Endpoint to uncompress the Credential Request after decryption. If absent then no compression algorithms are supported. The Wallet may use any of the supported compression algorithm to compress the Credential Request prior to encryption.
  * `encryption_required`: REQUIRED. Boolean value specifying whether the Credential Issuer requires the additional encryption on top of TLS for the Credential Requests. If the value is `true`, the Credential Issuer requires encryption for every Credential Request. If the value is `false`, the Wallet MAY choose whether it encrypts the request or not.
* `credential_response_encryption`: OPTIONAL. Object containing information about whether the Credential Issuer supports encryption of the Credential Response on top of TLS.
  * `alg_values_supported`: REQUIRED. A non-empty array containing a list of the JWE [@!RFC7516] encryption algorithms (`alg` values) [@!RFC7518] supported by the Credential Endpoint to encode the Credential Response in a JWT [@!RFC7519].
  * `enc_values_supported`: REQUIRED. A non-empty array containing a list of the JWE [@!RFC7516] encryption algorithms (`enc` values) [@!RFC7518] supported by the Credential Endpoint to encode the Credential Response in a JWT [@!RFC7519].
  * `zip_values_supported`: OPTIONAL. A non-empty array containing a list of the JWE [@!RFC7516] compression algorithms (`zip` values) [@!RFC7518] supported by the Credential Endpoint to compress the Credential Response prior to encryption. If absent then compression is not supported.
  * `encryption_required`: REQUIRED. Boolean value specifying whether the Credential Issuer requires the additional encryption on top of TLS for the Credential Response. If the value is `true`, the Credential Issuer requires encryption for every Credential Response and therefore the Wallet MUST provide encryption keys in the Credential Request. If the value is `false`, the Wallet MAY choose whether it provides encryption keys or not.
* `batch_credential_issuance`: OPTIONAL. Object containing information about the Credential Issuer's support for issuance of multiple Credentials in a batch in the Credential Endpoint. The presence of this parameter means that the issuer supports more than one key proof in the `proofs` parameter in the Credential Request so can issue more than one Verifiable Credential for the same Credential Dataset in a single request/response.
  * `batch_size`: REQUIRED. Integer value specifying the maximum array size for the `proofs` parameter in a Credential Request. It MUST be 2 or greater.
* `display`: OPTIONAL. A non-empty array of objects, where each object contains display properties of a Credential Issuer for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
  * `name`: OPTIONAL. String value of a display name for the Credential Issuer.
  * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
  * `logo`: OPTIONAL. Object with information about the logo of the Credential Issuer. Below is a non-exhaustive list of parameters that MAY be included:
    * `uri`: REQUIRED. String value that contains a URI where the Wallet can obtain the logo of the Credential Issuer. The Wallet needs to determine the scheme, since the URI value could use the `https:` scheme, the `data:` scheme, etc.
    * `alt_text`: OPTIONAL. String value of the alternative text for the logo image.
* `credential_configurations_supported`: REQUIRED. Object that describes specifics of the Credential that the Credential Issuer supports issuance of. This object contains a list of name/value pairs, where each name is a unique identifier of the supported Credential being described. This identifier is used in the Credential Offer as defined in (#credential-offer-parameters) to communicate to the Wallet which Credential is being offered. The value is an object that contains metadata about a specific Credential and contains the following parameters defined by this specification:
  * `format`: REQUIRED. A JSON string identifying the format of this Credential, i.e., `jwt_vc_json` or `ldp_vc`. Depending on the format value, the object contains further elements defining the type and (optionally) particular claims the Credential MAY contain and information about how to display the Credential. (#format-profiles) contains Credential Format Profiles introduced by this specification.
  * `scope`: OPTIONAL. A JSON string identifying the scope value that this Credential Issuer supports for this particular Credential. The value can be the same across multiple `credential_configurations_supported` objects. The Authorization Server MUST be able to uniquely identify the Credential Issuer based on the scope value. The Wallet can use this value in the Authorization Request as defined in (#credential-request-using-type-specific-scope). Scope values in this Credential Issuer metadata MAY duplicate those in the `scopes_supported` parameter of the Authorization Server. If `scope` is absent, the only way to request the Credential is using `authorization_details` [@!RFC9396] - in this case, the OAuth Authorization Server metadata for one of the Authorization Servers found from the Credential Issuer's Metadata must contain an `authorization_details_types_supported` that contains `openid_credential`.
  * `credential_signing_alg_values_supported`: OPTIONAL. A non-empty array of algorithm identifiers that identify the algorithms that the Issuer uses to sign the issued Credential. Algorithm identifier types and values used are determined by the Credential Format and are defined in (#format-profiles).
  * `cryptographic_binding_methods_supported`: OPTIONAL. A non-empty array of case sensitive strings that identify the representation of the cryptographic key material that the issued Credential is bound to, as defined in (#credential-binding). It MUST be present when Cryptographic Key Binding is required for a Credential, and omitted otherwise. If absent, Cryptographic Key Binding is not required for this credential. Support for keys in JWK format [@!RFC7517] is indicated by the value `jwk`. Support for keys expressed as a COSE Key object [@!RFC8152] (for example, used in [@!ISO.18013-5]) is indicated by the value `cose_key`. When the Cryptographic Key Binding method is a DID, valid values are a `did:` prefix followed by a method-name using a syntax as defined in Section 3.1 of [@!DID-Core], but without a `:` and method-specific-id. For example, support for the DID method with a method-name "example" would be represented by `did:example`.
  * `proof_types_supported`: OPTIONAL. Object that describes specifics of the key proof(s) that the Credential Issuer supports. It MUST be present if `cryptographic_binding_methods_supported` is present, and omitted otherwise. If absent, the Wallet is not required to supply proofs when requesting this credential. This object contains a list of name/value pairs, where each name is a unique identifier of the supported proof type(s). Valid values are defined in (#proof-types), other values MAY be used. The Wallet also uses this identifier in the Credential Request as defined in (#credential-request). The value in the name/value pair is an object that contains metadata about the key proof and contains the following parameters defined by this specification:
    * `proof_signing_alg_values_supported`: REQUIRED. A non-empty array of algorithm identifiers that the Issuer supports for this proof type. The Wallet uses one of them to sign the proof. Algorithm identifier types and values used are determined by the proof type and are defined in (#proof-types).
    * `key_attestations_required`: OPTIONAL. Object that describes the requirement for key attestations as described in (#keyattestation), which the Credential Issuer expects the Wallet to send within the proof(s) of the Credential Request. If the Credential Issuer does not require a key attestation, this parameter MUST NOT be present in the metadata. If both `key_storage` and `user_authentication` parameters are absent, the `key_attestations_required` parameter may be empty, indicating a key attestation is needed without additional constraints.
      * `key_storage`: OPTIONAL. A non-empty array defining values specified in (#keyattestation-apr) accepted by the Credential Issuer.
      * `user_authentication`: OPTIONAL. A non-empty array defining values specified in (#keyattestation-apr) accepted by the Credential Issuer.
  * `credential_metadata`: OPTIONAL. Object containing information relevant to the usage and display of issued Credentials. Credential Format-specific mechanisms can overwrite the information in this object to convey Credential metadata. Format-specific mechanisms, such as SD-JWT VC display metadata are always preferred by the Wallet over the information in this object, which serves as the default fallback. Below is a non-exhaustive list of parameters that MAY be included:
    * `display`: OPTIONAL. A non-empty array of objects, where each object contains the display properties of the supported Credential for a certain language. Below is a non-exhaustive list of parameters that MAY be included.
      * `name`: REQUIRED. String value of a display name for the Credential.
      * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. Multiple `display` objects MAY be included for separate languages. There MUST be only one object for each language identifier.
      * `logo`: OPTIONAL. Object with information about the logo of the Credential. The following non-exhaustive set of parameters MAY be included:
          * `uri`: REQUIRED. String value that contains a URI where the Wallet can obtain the logo of the Credential from the Credential Issuer. The Wallet needs to determine the scheme, since the URI value could use the `https:` scheme, the `data:` scheme, etc.
          * `alt_text`: OPTIONAL. String value of the alternative text for the logo image.
      * `description`: OPTIONAL. String value of a description of the Credential.
      * `background_color`: OPTIONAL. String value of a background color of the Credential represented as numerical color values defined in CSS Color Module Level 3 [@!CSS-Color].
      * `background_image`: OPTIONAL. Object with information about the background image of the Credential. At least the following parameter MUST be included:
          * `uri`: REQUIRED. String value that contains a URI where the Wallet can obtain the background image of the Credential from the Credential Issuer. The Wallet needs to determine the scheme, since the URI value could use the `https:` scheme, the `data:` scheme, etc.
      * `text_color`: OPTIONAL. String value of a text color of the Credential represented as numerical color values defined in CSS Color Module Level 3 [@!CSS-Color].
    * `claims`: OPTIONAL. A non-empty array of claims description objects as defined in (#claims-description-issuer-metadata).

An Authorization Server that only supports the Pre-Authorized Code grant type MAY omit the `response_types_supported` parameter in its metadata despite [@!RFC8414] mandating it.

Note: It can be challenging for a Credential Issuer that accepts tokens from multiple Authorization Servers to introspect an Access Token to check the validity and determine the permissions granted. Some ways to achieve this are relying on Authorization Servers that use [@!RFC9068] or by the Credential Issuer understanding the proprietary Access Token structures of the Authorization Servers.

Depending on the Credential Format, additional parameters might be present in the `credential_configurations_supported` object values, such as information about claims in the Credential. For Credential Format-specific claims, see the "Credential Issuer Metadata" subsections in (#format-profiles).

The Authorization Server MUST be able to determine from the Issuer metadata what claims are disclosed by the requested Credentials to be able to render meaningful End-User consent.

Additional Credential Issuer metadata parameters MAY be defined and used.
The Wallet MUST ignore any unrecognized parameters.

The following is a non-normative example of Credential Issuer metadata of a Credential in the IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format:

<{{examples/credential_metadata_sd_jwt_vc.json}}

Note: The Client MAY use other mechanisms to obtain information about the Verifiable Credentials that a Credential Issuer can issue.

See (#additional-issuer-metadata-examples) for additional examples of Credential Issuer Metadata.

## OAuth 2.0 Authorization Server Metadata {#as-metadata}

This specification also defines a new OAuth 2.0 Authorization Server metadata [@!RFC8414] parameter to publish whether the Authorization Server that the Credential Issuer relies on for authorization supports anonymous Token Requests with the Pre-Authorized Grant Type. It is defined as follows:

* `pre-authorized_grant_anonymous_access_supported`: OPTIONAL. A boolean indicating whether the Credential Issuer accepts a Token Request with a Pre-Authorized Code but without a `client_id`. The default is `false`.
* `interactive_authorization_endpoint`: OPTIONAL. URL of the Authorization Server's Interactive Authorization Endpoint. This URL MUST use the https scheme and MAY contain port, path, and query parameter components. If omitted, the Authorization Server does not support the Interactive Authorization Endpoint. If present, the Wallet SHOULD use this endpoint to obtain authorization as defined in (#interactive-authorization-endpoint).
* `require_interactive_authorization_request`: OPTIONAL. A boolean indicating whether the Authorization Server only accepts an Authorization Request for Credential issuance via the Interactive Authorization Endpoint defined in [#interactive-authorization-endpoint]. If omitted, the default value is `false`. This parameter MUST NOT be present if `interactive_authorization_endpoint` is omitted. Note that the presence of `interactive_authorization_endpoint` is sufficient for a Wallet to determine that it can use the Interactive Authorization Endpoint.

Additional Authorization Server metadata parameters MAY be defined and used,
as described in [@!RFC8414].
The Wallet MUST ignore any unrecognized parameters.

# Security Considerations {#security-considerations}

## Formal Security Analysis

The security properties of some features in a previous revision of this specification have been formally analyzed, see [@secanalysis].

## Best Current Practice for OAuth 2.0 Security {#securitybcp}

Implementers of this specification SHOULD follow the Best Current Practice for OAuth 2.0 Security given in [@BCP240]. It is RECOMMENDED that implementers do this by following the [@!FAPI2_Security_Profile] where it is applicable.

The parts of [@!FAPI2_Security_Profile] that may not be applicable when using this specification are:

1. Client authentication: The private_key_jwt and MTLS client authentication methods may not be practical to implement in a secure way for native app Wallets. The use of Wallet Attestations as defined in (#walletattestation) is RECOMMENDED instead.
2. Sender-constrained access tokens: MTLS sender constrained access token may not be practical to implement in a secure way for native app Wallets. The use of DPoP [@!RFC9449] is RECOMMENDED.
3. Pushed Authorization Requests: The Interactive Authorization Endpoint is permitted as an alternative to Pushed Authorization Requests (PAR), see (#interactive-authorization-endpoint).

## Trust between Wallet and Issuer

Credential Issuers often want to know what Wallet they are issuing Credentials to and how private keys are managed for the following reasons:

* The Credential Issuer MAY want to ensure that private keys are properly protected from exfiltration and replay to prevent an adversary from impersonating the legitimate Credential Holder by presenting their Credentials.
* The Credential Issuer MAY also want to ensure that the Wallet managing the Credentials adheres to certain policies and, potentially, was audited and approved under a certain regulatory and/or commercial scheme.

The following mechanisms in concert can be utilized to fulfill those objectives:

**Key attestation** is a mechanism where the key storage component or Wallet Provider asserts the cryptographic public keys and their security policy. The Wallet MAY provide this data in the Credential Request to allow the Credential Issuer to validate the cryptographic key management policy. This requires the Credential Issuer to rely on the trust anchor of the key attestation and the respective key management policy. While some existing platforms have key attestation formats, this specification introduces a common key attestation format that may be used by Credential Issuers for improved interoperability, see (#keyattestation).

**Client Authentication** enables a Wallet to authenticate with the Credential Issuer's Authorization Server. This authentication ensures that the Wallet complies with the Authorization Server's security, compliance, and governance standards, which may be required by trust frameworks, regulatory requirements, laws, or internal policies. Any method listed in the [OAuth Token Endpoint Authentication Methods IANA Registry](https://www.iana.org/assignments/oauth-parameters/oauth-parameters.xhtml#token-endpoint-auth-method) can be used for authentication during the Pushed Authorization or Token Request. The Authorization Server SHOULD specify its client authentication requirements using the `token_endpoint_auth_method` metadata parameter.

**Wallet Attestation** is a signed proof provided by the Wallet Provider, verifying the client's authenticity and genuineness. This process uses the mechanisms outlined in Attestation-Based Client Authentication, described in the Section [Wallet Attestation](#walletattestation). Once obtained, the Wallet Attestation can be used as a client authentication for the Wallet.

## Split-Architecture Wallets

A Wallet may consist of multiple components with varying levels of trust, security, and privacy. A common example of this is an architecture that involves both a server-side component and a native application. While the server component can offer advantages such as enhanced reliability, centralized security controls, and certain privacy protections, it also introduces distinct risks. These include increased difficulty in conducting audits (particularly by external security experts), the potential for opaque or unverified updates, and a higher susceptibility to insider threats.


To ensure the server component can provide meaningful functionality while preserving user privacy, the principle of data minimization is encouraged to be employed. In particular, ensure the confidentiality of user data, it is best to apply application level encryption to the Credential Request/Response, from the device through the server component.

It is important to note that when the server component acts as the trust anchor (e.g., for Wallet Attestations or Key Attestations), it cannot also serve as a safeguard against itself.

If the server component has access to authorization codes, pre-authorization codes, or other sensitive tokens or proofs, and no additional mitigations are implemented beyond those outlined here, it MAY be able to impersonate the application and gain access to confidential data. In cases where the server component is not fully trusted, these sensitive elements MUST NOT be passed to or routed through it.

## Credential Offer {#credential-offer-security}

The Wallet MUST consider the parameter values in the Credential Offer as not trustworthy, since the origin is not authenticated and the message integrity is not protected. The Credential Issuer is not considered trustworthy just because it sent the Credential Offer. Therefore, the Wallet MUST perform the same validation checks on the Credential Issuer as it would when initiating the flow directly from the Wallet. An attacker might attempt to use a Credential Offer to conduct a phishing or injection attack.

The Wallet MUST NOT accept Credentials just because this mechanism was used. All protocol steps defined in this specification MUST be performed in the same way as if the Wallet would have started the flow. 

The Credential Issuer MUST ensure the release of any privacy-sensitive data in Credential Offer is legal.

## Pre-Authorized Code Flow {#security-considerations-pre-authz-code}

### Replay Prevention

The Pre-Authorized Code Flow is vulnerable to the replay of the Pre-Authorized Code, because by design, it is not bound to a certain session (as the Authorization Code Flow does with PKCE). This means an attacker can replay the Pre-Authorized Code meant for a victim at another device. In a 'shoulder surfing' scenario, the attacker might scan the QR code while it is displayed on the victim's screen, and thereby get access to the Credential. As the Pre-Authorized Code is presented as a link or QR code, either may be shared beyond its intended use, allowing others to replay the transaction. Such replay attacks must be prevented using other means. The design facilitates the following options:

* Transaction Code: the Credential Issuer might set up a Transaction Code with the End-User (e.g., via text message or email) that needs to be presented in the Token Request.

### Transaction Code Phishing

An attacker might leverage the Credential issuance process and the End-User's trust in the Wallet to phish Transaction Codes sent out by a different service that grant the attacker access to services other than Credential issuance. The attacker could set up a Credential Issuer site and in parallel to the issuance request, trigger transmission of a Transaction Code to the End-User's phone from a service other than Credential issuance, e.g., from a payment service. The End-User would then be asked to enter this Transaction Code into the Wallet and since the Wallet sends this Transaction Code to the Token Endpoint of the Credential Issuer (the attacker), the attacker would get access to the Transaction Code, and access to that other service.

In order to cope with that issue, the Wallet is RECOMMENDED to interact with trusted Credential Issuers only. In that case, the Wallet would not process a Credential Offer with an untrusted issuer URL. The Wallet MAY also show the End-User the endpoint of the Credential Issuer it will be sending the Transaction Code to and ask the End-User for confirmation.

## Credential Lifecycle Management 

The Credential Issuer is supposed to be responsible for the lifecycle of its Credentials. This means the Credential Issuer will invalidate Credentials when it deems appropriate, e.g., if it detects fraudulent behavior.

The Wallet is supposed to detect signs of fraudulent behavior related to the Credential management in the Wallet (e.g., device rooting) and to act upon such signals. Options include Credential revocation at the Credential Issuer and/or invalidation of the key material used to cryptographically bind the Credential to the identifier of the End-User possessing that Credential.

## Proof replay {#key-proof-replay}

If an adversary obtains a key proof (as outlined in (#proof-types)), they could potentially replay that proof to a Credential Issuer to obtain a duplicate Credential or multiple duplicates issued to the same key. The `c_nonce` parameter serves as the main defense against this, by providing Credential Issuers with a mechanism to determine freshness of a generated proof and thus reject a proof when required. It is RECOMMENDED that Credential Issuers utilize the Nonce Endpoint as specified in (#nonce-endpoint) to protect against this. A Wallet can continue using a given nonce until it is rejected by the Credential Issuer. The Credential Issuer determines for how long a particular nonce can be used.

Note: For the attacker to be able to present a Credential bound to a replayed key proof to the Verifier, the attacker also needs to obtain the victim's private key. To limit this, Credential Issuers are RECOMMENDED to check how the Wallet protects the private keys, using mechanisms defined in (#keyattestation).

Note: To accommodate for clock offsets, the Credential Issuer server MAY accept proofs that carry an `iat` time in the reasonably near future (on the order of seconds or minutes). Because clock skews between servers and Clients may be large, servers MAY limit key proof lifetimes by using server-provided nonce values containing the time at the server rather than comparing the client-supplied `iat` time to the time at the server. Nonces created in this way yield the same result even in the face of arbitrarily large clock skews.

##  TLS Requirements

Implementations MUST follow [@!BCP195].
Whenever TLS is used, a TLS server certificate check MUST be performed, per [@!RFC6125].

## Protecting the Access Token

Access Tokens represent End-User authorization and consent to issue certain Credential(s). Long-lived Access Tokens giving access to Credentials MUST not be issued unless sender-constrained. Access Tokens with lifetimes longer than 5 minutes are, in general, considered long lived.

To sender-constrain Access Tokens, see the recommendations in (#securitybcp). If Bearer Access Tokens are stored by the Wallet, they MUST be stored in a secure manner, for example, encrypted using a key stored in a protected key store.

## Application-Layer Encryption {#encryption-security-considersations}

Depending on the architecture of the Wallet and the Issuer, adding encryption to requests and responses, beyond transport-level security (TLS), can offer enhanced data confidentiality. This can come at the cost of added complexity. This approach can be particularly relevant when the Wallet consists of multiple components with varying trust levels, such as a backend server and a client application. Similar considerations may apply to more complex Issuer architectures.

It is important that the Wallet component responsible for encryption can establish trust in the Issuer's key material to prevent man-in-the-middle attacks. The simplest way to achieve this is by retrieving the keys directly from the Issuer’s hosted Issuer Metadata. Alternatively, in the case of signed Issuer Metadata, the signature can be verified to ensure authenticity.

In cases where the application-layer encryption begins and terminates in the same component as TLS, it provides no additional protection.

While application-layer encryption can enhance the confidentiality of data in transit, it does not protect against other threats, such as access token theft, client impersonation, or other forms of unauthorized access. These risks must be mitigated through additional security measures.

# Implementation Considerations

## Claims-based Holder Binding of the Credential to the End-User possessing the Credential {#claims-based-binding}

Credentials not cryptographically bound to the identifier of the End-User possessing it (see (#credential-binding)), should be bound to the End-User possessing the Credential, based on the claims included in the Credential.

In Claims-based Holder Binding, no Cryptographic Key Binding material is provided. Instead, the issued Credential includes End-User claims that can be used by the Verifier to verify possession of the Credential by requesting presentation of existing forms of physical or digital identification that includes the same claims (e.g., a driving license or other ID cards in person, or an online ID verification service).

## Binding of the Credential without Cryptographic Key Binding or Claims-based Holder Binding {#no-binding}

Some Credential Issuers might choose to issue bearer Credentials without either Cryptographic Key Binding or Claims-based Holder Binding because they are meant to be presented without proof of possession.

One such use case is low assurance Credentials, such as coupons or tickets.

Another use case is when the Credential Issuer uses cryptographic schemes that can provide binding to the End-User possessing that Credential without explicit cryptographic material being supplied by the application used by that End-User. For example, in the case of the BBS Signature Scheme, the issued Credential itself is a secret and only a derivation from the Credential is presented to the Verifier. Effectively, the Credential is bound to the Credential Issuer's signature on the Credential, which becomes a shared secret transferred from the Credential Issuer to the End-User.

## Multiple Accesses to the Credential Endpoint

The Credential Endpoint can be accessed multiple times by a Wallet using the same Access Token, even for the same Credential. The Credential Issuer determines if the subsequent successful requests will return the same or an updated Credential, such as having a new expiration time or using the most current End-User claims.

The Credential Issuer MAY also decide to no longer accept the Access Token and a re-authentication or Token Refresh (see [@!RFC6749], Section 6) MAY be required at the Credential Issuer's discretion. The policies between the Credential Endpoint and the Authorization Server that MAY change the behavior of what is returned with a new Access Token are beyond the scope of this specification (see Section 7 of [@!RFC6749]).

The Credential Issuer SHOULD NOT revoke previously issued, valid Credentials solely as a result of a subsequent successful Credential Request. This, for example, ensures that the Wallet can keep a desired number of Credentials without causing additional revocation and issuance overhead.

The action leading to the Wallet performing another Credential Request can also be triggered by a background process, or by the Credential Issuer using an out-of-band mechanism (SMS, email, etc.) to inform the End-User.

## Relationship between the Credential Issuer Identifier in the Metadata and the Issuer Identifier in the Issued Credential

The Credential Issuer Identifier is always a URL using the `https` scheme, as defined in (#credential-issuer-identifier). Depending on the Credential Format, the Issuer Identifier in the issued Credential may not be a URL using the `https` scheme. Some other forms that it can take are a DID included in the `issuer` property in a [@VC_DATA] format, or the `Subject` value of the document signer certificate included in the `x5chain` element in an [@ISO.18013-5] format.

When the Issuer Identifier in the issued Credential is a DID, a non-exhaustive list of mechanisms the Credential Issuer MAY use to bind to the Credential Issuer Identifier is as follows:

1. Use the [@DIF.Well-Known_DID] Specification to provide binding between a DID and a certain domain.
1. If the Issuer Identifier in the issued Credential is an object, add to the object a `credential_issuer` claim, as defined in (#credential-issuer-identifier).

The Wallet MAY check the binding between the Credential Issuer Identifier and the Issuer Identifier in the issued Credential.

## Refreshing Issued Credentials

After a Verifiable Credential has been issued to the Holder, claim values about the subject of a Credential or a signature on the Credential may need to be updated. There are two possible mechanisms to do so.

First, the Wallet may receive an updated version of a Credential from a Credential Endpoint using a valid Access Token. This does not involve interaction with the End-User. If the Credential Issuer issued a Refresh Token to the Wallet, the Wallet would obtain a fresh Access Token by making a request to the Token Endpoint, as defined in Section 6 of [@!RFC6749].

Second, the Credential Issuer can reissue the Credential by starting the issuance process from the beginning. This would involve interaction with the End-User. A Credential needs to be reissued if the Wallet does not have a valid Access Token or a valid Refresh Token. With this approach, when a new Credential is issued, the Wallet might need to check if it already has a Credential of the same type and, if necessary, delete the old Credential. Otherwise, the Wallet might end up with more than one Credential of the same type, without knowing which one is the latest.

Credential Refresh can be initiated by the Wallet independently from the Credential Issuer, or the Credential Issuer can send a signal to the Wallet asking it to request Credential refresh. How the Credential Issuer sends such a signal is out of scope of this specification.

It is up to the Credential Issuer whether to update both the signature and the claim values, or only the signature.

## Batch Issuing Credentials

The Credential Issuer determines the number of the Credentials issued in the Credential Response, regardless of number of proofs/keys contained in the `proofs` parameter in the Credential Request.

## Pre-Final Specifications

Implementers should be aware that this specification uses several specifications that are not yet final specifications. Those specifications are:

* OpenID Federation 1.0 draft -43 [@!OpenID.Federation]
* SD-JWT-based Verifiable Credentials (SD-JWT VC) draft -11 [@!I-D.ietf-oauth-sd-jwt-vc]
* Attestation-Based Client Authentication draft -07 [@!I-D.ietf-oauth-attestation-based-client-auth]
* Token Status List draft -12 [@!I-D.ietf-oauth-status-list]

While breaking changes to the specifications referenced in this specification are not expected, should they occur, OpenID4VCI implementations should continue to use the specifically referenced versions above in preference to the final versions, unless updated by a profile or new version of this specification.

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

Claims containing time-related information, such as issuance or expiration dates, SHOULD be either individually randomized within an appropriate time window (e.g., within the last 24 hours), or rounded (e.g., to the start of the day), to avoid unintended correlation factors.

Credential Issuers specifically SHOULD discard values that can be used in collusion with a Verifier to track a user, such as the Issuer's signature or cryptographic key material to which an issued credential was bound to.

### Credential Offer

The Privacy Considerations in Section 11.2 of [@!RFC9101] apply to the `credential_offer` and
`credential_offer_uri` parameters defined in (#credential-offer).

### Authorization Request

The Wallet SHOULD NOT include potentially sensitive information in the Authorization Request,
for example, by including clear-text session information as a `state` parameter value or encoding
it in a `redirect_uri` parameter. A third party may observe such information through browser
history, etc. and correlate the user's activity using it.

### Wallet Attestation Subject {#walletattestation-sub}

The Wallet Attestation as defined in (#walletattestation) SHOULD NOT introduce a unique identifier specific to a single client.
The subject claim for the Wallet Attestation SHOULD be a value that is shared by all Wallet instances using this type of
wallet implementation. The value should be understood as an identifier of the Wallet type, rather than the specific Wallet
instance itself.

## Identifying the Credential Issuer

Information in the credential identifying a particular Credential Issuer, such as a Credential Issuer Identifier,
issuer's certificate, or issuer's public key may reveal information about the End-User.

For example, when a military organization or a drug rehabilitation center issues a vaccine
credential, verifiers can deduce that the owner of the Wallet storing such a Credential is a
military member or may have a substance use disorder.

In addition, when a Credential Issuer issues only one type of Credential, it might have privacy implications,
because if the Wallet has a Credential issued by that Issuer, its type and claim names can be
determined.

For example, if the National Cancer Institute only issued Credentials with cancer registry
information, it is possible to deduce that the owner of the Wallet storing such a Credential is a
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

<reference anchor="DID-Core" target="https://www.w3.org/TR/2022/REC-did-core-20220719/">
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

<reference anchor="VC_DATA" target="https://www.w3.org/TR/2022/REC-vc-data-model-20220303">
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

<reference anchor="VC_DATA_2.0" target="https://www.w3.org/TR/2025/REC-vc-data-model-2.0-20250515/">
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
    <author fullname="Gabe Cohen">
      <organization>Block</organization>
    </author>
   <date day="15" month="May" year="2025"/>
  </front>
</reference>

<reference anchor="VC_Data_Integrity" target="https://www.w3.org/TR/2025/REC-vc-data-integrity-20250515/">
  <front>
    <title>Verifiable Credential Data Integrity 1.0</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Ted Thibodeau Jr">
      <organization>OpenLink Software</organization>
    </author>
    <author fullname="Ivan Herman">
      <organization>W3C</organization>
    </author>
    <author fullname="Dave Longley">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Greg Bernstein">
      <organization>Invited Expert</organization>
    </author>
   <date day="15" month="May" year="2025"/>
  </front>
</reference>

<reference anchor="USASCII" target="https://www.unicode.org/L2/L2006/06388-review-incits4.pdf">
        <front>
          <title>Coded Character Set -- 7-bit American Standard Code for Information Interchange</title>
          <author>
            <organization>American National Standards Institute</organization>
          </author>
          <date year="2002"/>
        </front>
</reference>

<reference anchor="BCP195" target="https://www.rfc-editor.org/info/bcp195">
        <front>
          <title>BCP195</title>
          <author>
            <organization>IETF</organization>
          </author>
          <date month="Nov" year="2022"/>
        </front>
</reference>

<reference anchor="BCP240" target="https://www.rfc-editor.org/info/bcp240">
        <front>
          <title>BCP240</title>
          <author>
            <organization>IETF</organization>
          </author>
          <date month="Jan" year="2025"/>
        </front>
</reference>

<reference anchor="OpenID.Core" target="https://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 2</title>
    <author fullname="Nat Sakimura" initials="N." surname="Sakimura">
      <organization abbrev="NAT.Consulting (was at NRI)">NAT.Consulting</organization>
    </author>
    <author fullname="John Bradley" initials="J." surname="Bradley">
      <organization abbrev="Yubico (was at Ping Identity)">Yubico</organization>
    </author>
    <author fullname="Michael B. Jones" initials="M.B." surname="Jones">
      <organization abbrev="Self-Issued Consulting (was at Microsoft)">Self-Issued Consulting</organization>
    </author>
    <author fullname="Breno de Medeiros" initials="B." surname="de Medeiros">
      <organization abbrev="Google">Google</organization>
    </author>
    <author fullname="Chuck Mortimore" initials="C." surname="Mortimore">
      <organization abbrev="Disney (was at Salesforce)">Disney</organization>
    </author>
    <date day="15" month="December" year="2023"/>
  </front>
</reference>

<reference anchor="FAPI2_Security_Profile" target="https://openid.net/specs/fapi-security-profile-2_0.html">
  <front>
    <title>FAPI 2.0 Security Profile</title>
    <author initials="D." surname="Fett" fullname="Daniel Fett">
      <organization>Authlete</organization>
    </author>
    <author initials="D." surname="Tonge" fullname="Dave Tonge">
      <organization>Moneyhub Financial Technology Ltd.</organization>
    </author>
    <author initials="J." surname="Heenan" fullname="Joseph Heenan">
      <organization>Authlete</organization>
    </author>
   <date day="22" month="Feb" year="2025"/>
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

<reference anchor="CSS-Color" target="https://www.w3.org/TR/2022/REC-css-color-3-20220118/">
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

<reference anchor="JSON-LD" target="https://www.w3.org/TR/2020/REC-json-ld11-20200716/">
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
          <title>ISO/IEC 18013-5:2021 Personal identification — ISO-compliant driving licence — Part 5: Mobile driving licence (mDL) application</title>
          <author>
            <organization> ISO/IEC JTC 1/SC 17 Cards and security devices for personal identification</organization>
          </author>
          <date Month="September" year="2021"/>
        </front>
</reference>

<reference anchor="ISO.18045" target="https://www.iso.org/standard/72889.html">
        <front>
          <title>ISO/IEC 18045:2022 Information security, cybersecurity and privacy protection — Evaluation criteria for IT security — Methodology for IT security evaluation</title>
          <author>
            <organization>ISO</organization>
          </author>
          <date Month="August" year="2022"/>
        </front>
</reference>

<reference anchor="IANA.JOSE" target="https://www.iana.org/assignments/jose">
        <front>
          <title>JSON Object Signing and Encryption (JOSE)</title>
          <author>
            <organization>IANA</organization>
          </author>
        </front>
</reference>

<reference anchor="IANA.COSE" target="https://www.iana.org/assignments/cose/cose.xhtml">
        <front>
          <title>CBOR Object Signing and Encryption (COSE)</title>
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
          <organization>SPRIND</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>SPRIND</organization>
        </author>
        <author initials="D" surname="Fett" fullname="Daniel Fett">
          <organization>Authlete</organization>
        </author>
        <author initials="J" surname="Heenan" fullname="Joseph Heenan">
          <organization>Authlete</organization>
        </author>
       <date day="9" month="Jul" year="2025"/>
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

<reference anchor="OpenID.Federation" target="https://openid.net/specs/openid-federation-1_0-43.html">
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
          <date day="2" month="June" year="2025"/>
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

<reference anchor="IANA.URI.Schemes" target="https://www.iana.org/assignments/uri-schemes">
  <front>
    <title>Uniform Resource Identifier (URI) Schemes</title>
    <author>
      <organization>IANA</organization>
    </author>
    <date/>
  </front>
</reference>

<reference anchor="IANA.Well-Known.URIs" target="https://www.iana.org/assignments/well-known-uris">
  <front>
    <title>Well-Known URIs</title>
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

<reference anchor="secanalysis" target="https://elib.uni-stuttgart.de/items/07055a8e-a85e-42b9-98b5-11f046d5fb91">
  <front>
    <title>OpenID for Verifiable Credentials: Formal Security Analysis using the Web Infrastructure Model</title>
    <author fullname="Fabian Hauck">
    </author>
    <date day="2" month="October" year="2023"/>
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

### VC Signed as a JWT, Not Using JSON-LD {#jwt-vc-json}

#### Format Identifier

The Credential Format Identifier is `jwt_vc_json`.

When the `format` value is `jwt_vc_json`, the entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules.

#### Credential Issuer Metadata {#server-metadata-jwt-vc-json}

Cryptographic algorithm identifiers used in the `credential_signing_alg_values_supported` parameter are case sensitive strings and SHOULD be one of those JWS Algorithm Names defined in [@IANA.JOSE].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters).

* `credential_definition`: REQUIRED. Object containing the detailed description of the Credential type. It consists of the following parameter:
  * `type`: REQUIRED. Array designating the types a certain Credential type supports, according to [@VC_DATA], Section 4.3.

The following is a non-normative example of an object containing the `credential_configurations_supported` parameter for Credential Format `jwt_vc_json`:

<{{examples/credential_metadata_jwt_vc_json.json}}

#### Authorization Details {#authorization-jwt-vc-json}

The following is a non-normative example of an authorization details object with Credential Format `jwt_vc_json`:

<{{examples/authorization_details_jwt_vc_json.json}}

#### Credential Response {#credential-response-jwt-vc-json}

The value of the `credential` claim in the Credential Response MUST be a JWT. Credentials of this format are already a sequence of base64url-encoded values separated by period characters and MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response with Credential Format `jwt_vc_json` (with line breaks within values for display purposes only):

<{{examples/credential_response_jwt_vc_json.txt}}

The following is the dereferenced document for the Issuer HTTP URL identifier that matches the Credential in the above example (with line breaks within values for display purposes only):

<{{examples/issuer_jwks.json}}

#### Interactive Authorization Endpoint Binding {#iae-binding-jwt-vc-json}

To bind the Interactive Authorization Endpoint to a Verifiable Presentation using the Credential Format defined in this section, the `aud` claim value MUST be set to the Interactive Authorization Endpoint, prefixed with `iae:` (e.g., `iae:https://example.com/iae`).

### VC Secured using Data Integrity, using JSON-LD, with a Proof Suite Requiring Linked Data Canonicalization

#### Format Identifier

The Credential Format Identifier is `ldp_vc`.

When the `format` value is `ldp_vc`, the entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules. 

The `@context` value in the `credential_definition` object can be used by the Wallet to check whether it supports a certain VC or not. If necessary, the Wallet could apply JSON-LD processing to the Credential issued by the Credential Issuer.

Note: Data Integrity used to be called Linked Data Proofs, hence the "ldp" in the Credential Format Identifier.

#### Credential Issuer Metadata {#server-metadata-ldp-vc}

Cryptographic algorithm identifiers used in the `credential_signing_alg_values_supported` parameter are case sensitive strings and SHOULD be one of those signature suite identifiers defined in [@LD_Suite_Registry].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters):

* `credential_definition`: REQUIRED. Object containing the detailed description of the Credential type. It consists of the following parameters:
  * `@context`: REQUIRED. Array as defined in [@VC_DATA], Section 4.1.
  * `type`: REQUIRED. Array designating the types a certain credential type supports, according to [@VC_DATA], Section 4.3.

The following is a non-normative example of an object containing the `credential_configurations_supported` parameter for Credential Format `ldp_vc`:

<{{examples/credential_metadata_ldp_vc.json}}

#### Authorization Details {#authorization-ldp-vc}

The following is a non-normative example of an authorization details object with Credential Format `ldp_vc`:

<{{examples/authorization_details_ldp_vc.json}}

#### Credential Response

The value of the `credential` claim in the Credential Response MUST be a JSON object. Credentials of this format MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response with Credential Format `ldp_vc`:

<{{examples/credential_response_ldp_vc.txt}}

#### Interactive Authorization Endpoint Binding {#iae-binding-ldp-vc}

To bind the Interactive Authorization Endpoint to a Verifiable Presentation using the Credential Format defined in this section, the `domain` claim value MUST be set to the Interactive Authorization Endpoint, prefixed with `iae:` (e.g., `iae:https://example.com/iae`).

### VC signed as a JWT, Using JSON-LD

#### Format Identifier

The Credential Format Identifier is `jwt_vc_json-ld`.

When the `format` value is `jwt_vc_json-ld`, the entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules.

The `@context` value in the `credential_definition` parameter can be used by the Wallet to check whether it supports a certain VC or not. If necessary, the Wallet could apply JSON-LD processing to the Credential issued by the Credential Issuer.

#### Credential Issuer Metadata

The definitions in (#server-metadata-ldp-vc) apply for metadata of Credentials of this type as well. 

#### Authorization Details

The definitions in (#authorization-ldp-vc) apply for Credentials of this type as well.

#### Credential Response

The definitions in (#credential-response-jwt-vc-json) apply for Credentials of this type as well.

#### Interactive Authorization Endpoint Binding

The definitions in (#iae-binding-jwt-vc-json) apply to the Credentials of this format.

## Mobile Documents or mdocs (ISO/IEC 18013) {#mdocs}

This section defines a Credential Format Profile for credentials that conform to the mobile document (mdoc) format specified in ISO/IEC 18013-5 [@ISO.18013-5].

ISO/IEC 18013-5:2021 [@ISO.18013-5] defines the mdoc format in the context of mobile driving licences (mDLs). While the specification is focused on mDLs, the mdoc format itself is general-purpose and can be used for other types of Credentials.

### Format Identifier

The Credential Format Identifier is `mso_mdoc`. This refers to the Mobile Security Object (MSO) which secures the mdoc data model encoded as CBOR.

### Credential Issuer Metadata {#server-metadata-mso-mdoc}

Cryptographic algorithm identifiers used in the `credential_signing_alg_values_supported` parameter correspond to the numeric COSE algorithm identifiers used to secure the `IssuerAuth` COSE structure, as defined in [@!ISO.18013-5]. The values SHOULD be chosen from the following:

* The value exactly matches the `alg` value in the `IssuerAuth` COSE header;
* Or, the value is a fully specified algorithm, as defined in [@!I-D.ietf-jose-fully-specified-algorithms], and the combination of the `alg` value and the signing key's curve in the `IssuerAuth` COSE structure matches the combination specified by the fully specified algorithm.

Consequently, depending on the approach taken by the Issuer as described above, the `alg` value in the `IssuerAuth` COSE header MAY differ from the identifier listed in the `credential_signing_alg_values_supported` metadata parameter. This metadata parameter is primarily informational for the Wallet.

Example:
If the `IssuerAuth` structure contains an `alg` header value of `-7` (ECDSA with SHA-256, per [@!IANA.COSE]) and is signed using a P-256 key, it matches both `-7` and `-9` in `credential_signing_alg_values_supported`. The latter (`-9`) corresponds to ECDSA with P-256 and SHA-256, as defined in [@!I-D.ietf-jose-fully-specified-algorithms].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters).

* `doctype`: REQUIRED. String identifying the Credential type, as defined in [@!ISO.18013-5].

The following is a non-normative example of an object containing the `credential_configurations_supported` parameter for Credential Format `mso_mdoc`:

<{{examples/credential_metadata_mso_mdoc.json}}

### Authorization Details

The following is a non-normative example of an authorization details object with Credential Format `mso_mdoc`:

<{{examples/authorization_details_mso_doc.json}}

### Credential Response

The value of the `credential` claim in the Credential Response MUST be a string that is the base64url-encoded representation of the CBOR-encoded `IssuerSigned` structure, as defined in [@!ISO.18013-5].

The following is a non-normative example of a Credential Response containing a Credential of format `mso_mdoc`.

<{{examples/credential_response_mso_mdoc.txt}}

### Interactive Authorization Endpoint Binding {#iae-binding-mso-mdoc}

To bind the Interactive Authorization Endpoint to a Verifiable Presentation using the Credential Format defined in this section, the `SessionTranscript` CBOR structured as defined in Section 9.1.5.1 in [@ISO.18013-5] MUST be used in Verifiable Presentations submitted in a response to Interactive Authorization Requests using the `openid4vp_presentation` interaction type, with the following modifications. This `SessionTranscript` differs from those defined in Section B.5.6 in [@OpenID4VP] and is defined as follows:

* `DeviceEngagementBytes` MUST be `null`.
* `EReaderKeyBytes` MUST be `null`.
* `Handover` MUST be the `OpenID4VCIIAEHandover` CBOR structure as defined below.

Note: The following section contains a definition in Concise Data Definition Language (CDDL), a language used to define data structures - see [@RFC8610] for more details. `bstr` refers to Byte String, defined as major type 2 in CBOR and `tstr` refers to Text String, defined as major type 3 in CBOR (encoded in utf-8) as defined in section 3.1 of [@RFC8949].

```cddl
OpenID4VCIIAEHandover = [
  "OpenID4VCIIAEHandover", ; A fixed identifier for this handover type
  OpenID4VCIIAEHandoverInfoHash ; A cryptographic hash of OpenID4VCIIAEHandoverInfo
]

; Contains the sha-256 hash of OpenID4VCIIAEHandoverInfoBytes
OpenID4VCIIAEHandoverInfoHash = bstr

; Contains the bytes of OpenID4VCIIAEHandoverInfo encoded as CBOR
OpenID4VCIIAEHandoverInfoBytes = bstr .cbor OpenID4VCIIAEHandoverInfo

OpenID4VCIIAEHandoverInfo = [
  iae,
  nonce,
  jwkThumbprint
] ; Array containing handover parameters

iae = tstr

nonce = tstr

jwkThumbprint = bstr
```

The `OpenID4VCIIAEHandover` structure has the following elements:

* The first element MUST be the string `OpenID4VCIIAEHandover`. This serves as a unique identifier for the handover structure to prevent misinterpretation or confusion.
* The second element MUST be a Byte String which contains the sha-256 hash of the bytes of `OpenID4VCIIAEHandoverInfo` when encoded as CBOR.
* The `OpenID4VCIIAEHandoverInfo` has the following elements:
  * The first element MUST be the string representing the Interactive Authorization Endpoint of the request as described in (#interactive-authorization-request). It MUST NOT be prefixed with `iae:`.
  * The second element MUST be the value of the `nonce` request parameter.
  * For the Response Mode `iae_post.jwt`, the third element MUST be the JWK SHA-256 Thumbprint as defined in [@!RFC7638], encoded as a Byte String, of the Verifier's public key used to encrypt the response. If the Response Mode is `iae_post`, the third element MUST be `null`. For unsigned requests, including the JWK Thumbprint in the `SessionTranscript` allows the Verifier to detect whether the response was re-encrypted by a third party, potentially leading to the leakage of sensitive information. While this does not prevent such an attack, it makes it detectable and helps preserve the confidentiality of the response.  

The following is a non-normative example of the input JWK for calculating the JWK Thumbprint in the context of `OpenID4VCIIAEHandoverInfo`:

```json
{
  "kty": "EC",
  "crv": "P-256",
  "x": "DxiH5Q4Yx3UrukE2lWCErq8N8bqC9CHLLrAwLz5BmE0",
  "y": "XtLM4-3h5o3HUH0MHVJV0kyq0iBlrBwlh8qEDMZ4-Pc",
  "use": "enc",
  "alg": "ECDH-ES",
  "kid": "1"
}
```

The following is a non-normative example of the `OpenID4VCIIAEHandoverInfo` structure:

```
Hex:

837768747470733a2f2f6578616d706c652e636f6d2f696165782b6578633767426b
786a7831726463397564527276654b7653734a4971383061766c58654c4868477771
744158204283ec927ae0f208daaa2d026a814f2b22dca52cf85ffa8f3f8626c6bd66
9047

CBOR diagnostic:

83                                   # array(3)
  77                                 #   string(23)
    68747470733a2f2f6578616d706c652e #     "https://example."
    636f6d2f696165                   #     "com/iae"
  78 2b                              #   string(43)
    6578633767426b786a78317264633975 #     "exc7gBkxjx1rdc9u"
    64527276654b7653734a497138306176 #     "dRrveKvSsJIq80av"
    6c58654c48684777717441           #     "lXeLHhGwqtA"
  58 20                              #   bytes(32)
    4283ec927ae0f208daaa2d026a814f2b #     "B\x83ì\x92zàò\x08Úª-\x02j\x81O+"
    22dca52cf85ffa8f3f8626c6bd669047 #     ""Ü¥,ø_ú\x8f?\x86&Æ½f\x90G"
```

The following is a non-normative example of the `OpenID4VCIIAEHandover` structure:

```
Hex:

82754f70656e49443456434949414548616e646f7665725820df679426cc1bf8996e
8eb549ee078815a87a97c5e95c1c5a8ec39eedca28a838

CBOR diagnostic:

82                                   # array(2)
  75                                 #   string(21)
    4f70656e49443456434949414548616e #     "OpenID4VCIIAEHan"
    646f766572                       #     "dover"
  58 20                              #   bytes(32)
    df679426cc1bf8996e8eb549ee078815 #     "ßg\x94&Ì\x1bø\x99n\x8eµIî\x07\x88\x15"
    a87a97c5e95c1c5a8ec39eedca28a838 #     "¨z\x97Åé\\x1cZ\x8eÃ\x9eíÊ(¨8"
```

The following is a non-normative example of the `SessionTranscript` structure:

```
Hex:

83f6f682754f70656e49443456434949414548616e646f7665725820df679426cc1b
f8996e8eb549ee078815a87a97c5e95c1c5a8ec39eedca28a838

CBOR diagnostic:

83                                   # array(3)
  f6                                 #   null
  f6                                 #   null
  82                                 #   array(2)
    75                               #     string(21)
      4f70656e4944345643494941454861 #       "OpenID4VCIIAEHa"
      6e646f766572                   #       "ndover"
    58 20                            #     bytes(32)
      df679426cc1bf8996e8eb549ee0788 #       "ßg\x94&Ì\x1bø\x99n\x8eµIî\x07\x88"
      15a87a97c5e95c1c5a8ec39eedca28 #       "\x15¨z\x97Åé\\x1cZ\x8eÃ\x9eíÊ("
      a838                           #       "¨8"
```

## IETF SD-JWT VC

This section defines a Credential Format Profile for Credentials complying with [@!I-D.ietf-oauth-sd-jwt-vc].

### Format Identifier

The Credential Format Identifier is `dc+sd-jwt`.

### Credential Issuer Metadata {#server-metadata-sd-jwt-vc}

Cryptographic algorithm identifiers used in the `credential_signing_alg_values_supported` parameter are case sensitive strings and SHOULD be one of those JWS Algorithm Names defined in [@IANA.JOSE].

The following additional Credential Issuer metadata parameters are defined for this Credential Format for use in the `credential_configurations_supported` parameter, in addition to those defined in (#credential-issuer-parameters).


* `vct`: REQUIRED. String designating the type of the Credential, as defined in [@!I-D.ietf-oauth-sd-jwt-vc].

The following is a non-normative example of an object comprising the `credential_configurations_supported` parameter for Credential Format `dc+sd-jwt`.

<{{examples/credential_metadata_sd_jwt_vc.json}}

### Authorization Details {#authorization-sd-jwt-vc}

The following is a non-normative example of an authorization details object with Credential Format `dc+sd-jwt`.

<{{examples/authorization_details_sd_jwt_vc.json}}

### Credential Response {#credential-response-sd-jwt-vc}

The value of the `credential` claim in the Credential Response MUST be a string that is an SD-JWT VC. Credentials of this format are already suitable for transfer and, therefore, they need not and MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response containing a Credential of format `dc+sd-jwt` (with line breaks within values for display purposes only).

<{{examples/credential_response_sd_jwt_vc.txt}}

### Interactive Authorization Endpoint Binding {#iae-binding-sd-jwt-vc}

To bind the Interactive Authorization Endpoint to a Verifiable Presentation using the Credential Format defined in this section, the `aud` claim in the Key Binding JWT MUST be set to the derived Origin (as defined in (#iae-require-presentation)) of the Interactive Authorization Endpoint, prefixed with `iae:` (e.g., `iae:https://example.com`).

# Claims Description 

Claims description objects are used in two places in this specification, in the
Credential Issuer metadata and in the Authorization Details. The following
sections define the structure and semantics of claims description objects.

## Claims Description for Authorization Details {#claims-description-authorization-details}

A claims description object as used in authorization details is an object that defines the
requirements for the claims that the Wallet requests to be included in the
Credential.

For a claims description object, the following keys defined by this
specification can be used to describe the claim or claims (other keys MAY also
be used):

  * `path`: REQUIRED. Non-empty array. Claim path pointer as defined in (#claims_path_pointer)
    to identify the claim(s) in the Credential.
  * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the
    Wallet will only accept a Credential that includes this claim. If set to
    `false`, the claim is not required to be included in the Credential. If the
    `mandatory` parameter is omitted, the default value is `false`.

The rules defined in (#claims-description-processing) apply.

## Claims Description for Issuer Metadata {#claims-description-issuer-metadata}

A claims description object as used in the Credential Issuer metadata is an
object used to describe how a certain claim in the Credential is
displayed to the End-User. It is used in the `claims`
parameter in the Credential Issuer metadata defined in (#format-profiles). The
following keys can be used to describe the claim or claims:

  * `path`: REQUIRED. The value MUST be a non-empty array representing a claims 
    path pointer that specifies the path to a claim within the credential, as
    defined in (#claims_path_pointer).
  * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the
    Credential Issuer will always include this claim in the issued Credential.
    If set to `false`, the claim is not included in the issued Credential if the
    wallet did not request the inclusion of the claim, and/or if the Credential
    Issuer chose to not include the claim. If the `mandatory` parameter is
    omitted, the default value is `false`.
  * `display`: OPTIONAL. A non-empty array of objects, where each object contains display
    properties of a certain claim in the Credential for a certain language.
    Below is a non-exhaustive list of valid parameters that MAY be included:
     * `name`: OPTIONAL. String value of a display name for the claim.
     * `locale`: OPTIONAL. String value that identifies the language of this object,
       represented as language tag values defined in BCP47 [@!RFC5646]. There
       MUST be only one object for each language identifier.

The rules defined in (#claims-description-processing) apply.

## Processing Rules for Claims Description Objects {#claims-description-processing}

The order of claims description objects in the `claims`
array is used by the Wallet to determine the order in which the claims are
displayed to the End-User, unless another mechanism is defined by the profile.

When a repeated or contradictory claim description is provided, the processing
MUST be aborted. This is in particular the case if 

 - the same claim is addressed by two or more claims description objects in the
   `claims` array, or
 - there is a claims description object with a `path` that addresses a set of
   claims in an array (using `null`, as defined in (#claims_path_pointer)) and
   another object that uses a non-negative integer to address a specific claim
   in the same array, or
 - there is a claims description object indicating that a certain claim is an array
   (implied by using `null` or a non-negative integer in the `path`), and another
   object indicating that the same claim is an object (implied by using a string
   in the `path`).


# Claims Path Pointer {#claims_path_pointer}

A claims path pointer is a pointer into the Verifiable Credential, identifying one or more claims.
A claims path pointer MUST be a non-empty array of strings, nulls and integers.
A claims path pointer can be processed, which means it is applied to a credential. The results of
processing are the referenced claims.

## Semantics for JSON-based credentials

This section defines the semantics of a claims path pointer when applied to a JSON-based credential.

A string value indicates that the respective key is to be selected, a null value
indicates that all elements of the currently selected array(s) are to be selected;
and a non-negative integer indicates that the respective index in an array is to be selected.
Negative integer values are not used for JSON-based credentials. The path
is formed as follows:

Start with an empty array and repeat the following until the full path is formed. 

  - To address a particular claim within an object, append the key (claim name)
    to the array.
  - To address an element within an array, append the index to the array (as a
    non-negative, 0-based integer).
  - To address all elements within an array, append a null value to the array.

### Processing

In detail, the array is processed from left to right as follows:

 1. Select the root element of the Credential, i.e., the top-level object of the
    JSON structure representing the claims in the Credential.
 2. Process the query of the claims path pointer array from left to right:
    1. If the path component is a string, select the element in the respective
       key in the currently selected element(s). If any of the currently
       selected element(s) is not an object, abort processing and return an
       error. If the key does not exist in any element currently selected,
       remove that element from the selection.
    2. If the path component is null, select all elements of the currently
       selected array(s). If any of the currently selected element(s) is not an
       array, abort processing and return an error.
    3. If the path component is a negative integer, abort processing and return an error.
    4. If the path component is a non-negative integer, select the element at
       the respective index in the currently selected array(s). If any of the
       currently selected element(s) is not an array, abort processing and
       return an error. If the index does not exist in a selected array, remove
       that array from the selection.
  3. If the set of elements currently selected is empty, abort processing and
     return an error.

The result of the processing is the set of selected JSON elements.

## Semantics for ISO mdoc-based credentials

This section defines the semantics of a claims path pointer when applied to a
credential in ISO mdoc format.

A claims path pointer into an ISO mdoc always contains at least two path components of type string.
Start with an array of two elements where the first element represents
the namespace and the second element represents the data element identifier in that namespace
and repeat the following until the full path is formed.
  * To address a claim within a data structure (i.e., map), append the corresponding key (string or integer).
  * To address an element within an array, append the index (as a non-negative, 0-based integer).
  * To address all elements of an array, append a null value.

Whether a non-negative integer is interpreted as a map key or an array index depends on the type of the
currently selected element.

### Processing

In detail, the array is processed from left to right as follows:

1. If the path contains fewer than two path components, abort processing and return an error.
2. Select the namespace referenced by the first path component. If the namespace does not exist in the mdoc, abort processing and return an error.
3. Select the data element with the data element identifier that matches the second path component. If the data element does not exist in the mdoc, abort processing and return an error.
4. Process each subsequent path component as follows:
   1. If all the currently selected element(s) are arrays, and the path component is a non-negative integer, select the element at the given index in each array. If the index is out of bounds in a selected array, remove that array from the selection.
   2. If all the currently selected element(s) are maps, and the path component is a string or integer, select the element(s) associated with the key. If the key does not exist in a selected map, remove that map from the selection.
   3. If the path component is null, select all elements in each currently selected array. If any selected element is not an array, abort processing and return an error.
   4. If none of the above applied, abort processing and return an error.
5. If the set of elements currently selected is empty, abort processing and return an error.

The result of the processing is the set of selected elements contained within the selected data element value, or the value itself.

## Claims Path Pointer Example {#claims_path_pointer_example}

The following shows a non-normative, simplified example of a JSON-based Credential:

```json
{
  "name": "Arthur Dent",
  "address": {
    "street_address": "42 Market Street",
    "locality": "Milliways",
    "postal_code": "12345"
  },
  "degrees": [
    {
      "type": "Bachelor of Science",
      "university": "University of Betelgeuse"
    },
    {
      "type": "Master of Science",
      "university": "University of Betelgeuse"
    }
  ],
  "nationalities": ["British", "Betelgeusian"]
}
```

The following shows examples of claims path pointers and the respective selected
claims:

- `["name"]`: The claim `name` with the value `Arthur Dent` is selected.
- `["address"]`: The claim `address` with its sub-claims as the value is
  selected.
- `["address", "street_address"]`: The claim `street_address` with the value `42
  Market Street` is selected.
- `["degrees", null, "type"]`: All `type` claims in the `degrees` array are
  selected.
- `["nationalities", 1]`: The second nationality is selected.

# Key Attestations {#keyattestation}

A key attestation, as defined by this specification, is a verifiable statement that demonstrates the authenticity and security properties of a key and its storage component to the Credential Issuer. Keys can be stored in various key storage components, which vary in their ability to protect the private key from extraction and duplication, as well as in the methods used for User authentication to enable key operations. These key storage components can be software-based or hardware-based and may reside on the same device as the Wallet, on external security tokens, or on remote services that facilitate cryptographic key operations. Key attestations are issued either by the Wallet's key storage component itself or by the Wallet Provider. When the Wallet Provider creates the key attestation, it MUST verify the authenticity of its claims about the keys, possibly using platform-specific key attestations.

A Wallet MAY provide key attestations to inform the Credential Issuer about the properties of the cryptographic public keys, such as those used in proof types sent in the Credential Request. Credential Issuers SHOULD evaluate these key attestations to ensure the cryptographic keys meet their security requirements, based on the trust framework, regulatory requirements, laws, or internal policies. A Credential Issuer MUST communicate the need to evaluate key attestations through its metadata or via an out-of-band mechanism.

Key attestations are used by various Credential Issuers with different trust frameworks and requirements, so a common approach is needed for interoperability. Therefore, key attestations SHOULD follow a common format. This helps Credential Issuers create consistent evaluation processes, reducing complexity and errors. Common formats also simplify compliance with regulatory requirements across jurisdictions and support the creation of shared best practices and security standards.

## Key Attestation in JWT format {#keyattestation-jwt}

The JWT is signed by the Wallet Provider or the Wallet's key storage component itself and contains the following elements:

* in the JOSE header,
  * `alg`: REQUIRED. A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry [@IANA.JOSE]. It MUST NOT be `none` or an identifier for a symmetric algorithm (MAC).
  * `typ`: REQUIRED. MUST be `key-attestation+jwt`, which explicitly types the key proof JWT as recommended in Section 3.11 of [@!RFC8725].

The key attestation may use `x5c`, `kid` or `trust_chain` (as defined in (#jwt-proof-type)) to convey the public key and the associated trust mechanism to sign the key attestation.

* in the JWT body,
  * `iat`: REQUIRED (number). Integer for the time at which the key attestation was issued using the syntax defined in [@!RFC7519].
  * `exp`: OPTIONAL (number). Integer for the time at which the key attestation and the key(s) it is attesting expire, using the syntax defined in [@!RFC7519]. MUST be present if the attestation is used with the JWT proof type.
  * `attested_keys` : REQUIRED. A non-empty array of attested keys from the same key storage component using the syntax of JWK as defined in [@!RFC7517].
  * `key_storage` : OPTIONAL. A non-empty array of case sensitive strings that assert the attack potential resistance of the key storage component and its keys attested in the `attested_keys` parameter. This specification defines initial values in (#keyattestation-apr).
  * `user_authentication` : OPTIONAL. A non-empty array of case sensitive strings that assert the attack potential resistance of the user authentication methods allowed to access the private keys from the `attested_keys` parameter. This specification defines initial values in (#keyattestation-apr).
  * `certification` : OPTIONAL. A String that contains a URL that links to the certification of the key storage component.
  * `nonce`: OPTIONAL. String that represents a nonce provided by the Issuer to prove that a key attestation was freshly generated.
  * `status`: OPTIONAL. JSON Object representing the supported revocation check mechanisms, such as the one defined in [@!I-D.ietf-oauth-status-list]

If used with the `jwt` proof type, the Credential Issuer MUST validate that the JWT used as a proof is signed by a key contained in the attestation in the JOSE Header.

This is an example of a Key Attestation:

```json
{
  "typ": "key-attestation+jwt",
  "alg": "ES256",
  "x5c": ["MIIDQjCCA..."]
}
.
{
  "iss": "<identifier of the issuer of this key attestation>",
  "iat": 1516247022,
  "exp": 1541493724,
  "key_storage": [ "iso_18045_moderate" ],
  "user_authentication": [ "iso_18045_moderate" ],
  "attested_keys": [
    {
      "kty": "EC",
      "crv": "P-256",
      "x": "TCAER19Zvu3OHF4j4W4vfSVoHIP1ILilDls7vCeGemc",
      "y": "ZxjiWWbZMQGHVWKVQ4hbSIirsVfuecCE6t4jT9F2HZQ"
    }
  ]
}
```

## Attack Potential Resistance {#keyattestation-apr}

This specification defines the following values for `key_storage` and `user_authentication`:

* `iso_18045_high`: It MUST be used when key storage or user authentication is resistant to attack with attack potential "High", equivalent to VAN.5 according to [@!ISO.18045].
* `iso_18045_moderate`: It MUST be used when key storage or user authentication is resistant to attack with attack potential "Moderate", equivalent to VAN.4 according to [@!ISO.18045].
* `iso_18045_enhanced-basic`: It MUST be used when key storage or user authentication is resistant to attack with attack potential "Enhanced-Basic", equivalent to VAN.3 according to [@!ISO.18045].
* `iso_18045_basic`: It MUST be used when key storage or user authentication is resistant to attack with attack potential "Basic", equivalent to VAN.2 according to [@!ISO.18045].

Specifications that extend this list MUST choose collision-resistant values.

When [@!ISO.18045] is not used, ecosystems may define their own values. If the value does not map to a well-known specification, it is RECOMMENDED that the value is a URL that gives further information about the attack potential resistance and possible relations to level of assurances.

# Wallet Attestations in JWT format {#walletattestation}

The Wallet Attestation defined in this section is a client authentication method especially designed for native App Wallets. Instead of using platform-specific key, app, and/or device attestations directly, it uses a key-bound, platform agnostic common format based on JWTs. This allows Authorization Servers to authenticate Wallets across different platforms in a unified fashion and exposes only a minimum dataset. Also, it allows the Wallet app to directly interact with the Credential Issuer without involving the Wallet Provider's backend. The Authorization Server MUST verify that the Wallet Attestation is signed by an issuer that the Credential Issuer trusts for this purpose.

In a typical architecture of a native Wallet App, the Wallet Provider's backend will use attestations provided by the mobile operating system, like iOS's DeviceCheck or Android's Play Integrity, to validate the app's integrity and authenticity before issuing the Wallet Attestation.

There are two requests where Wallets may need to authenticate during Credential issuance:

- The Wallet sends it in the Pushed Authorization Request
- The Wallet sends it in the Token Request

The Wallet Attestation format follows Section 5.1 "Client Attestation JWT" of [@!I-D.ietf-oauth-attestation-based-client-auth]. The Wallet Attestation additionally includes the following JWT Claims:

* `wallet_name`: OPTIONAL. String containing a human-readable name of the Wallet.
* `wallet_link`: OPTIONAL. String containing a URL to get further information about the Wallet and the Wallet Provider.
* `status`: OPTIONAL. Status mechanism for the Wallet Attestation as defined in [@!I-D.ietf-oauth-status-list]

The following is a non-normative example of a decoded Wallet Attestation:

```
{
  "typ": "oauth-client-attestation+jwt",
  "alg": "ES256",
  "kid": "11"
}
.
{
  "iss": "https://wallet-provider.example.com",
  "sub": "https://wallet.example.org",
  "wallet_name": "Wallet Solution X by Wonderland State Department",
  "wallet_link": "https://example.com/wallet/detail_info.html",
  "nbf": 1300815780,
  "exp": 1300819380,
  "cnf": {
    "jwk": {
      "kty": "EC",
      "use": "sig",
      "crv": "P-256",
      "x": "18wHLeIgW9wVN6VD1Txgpqy2LszYkMf6J8njVAibvhM",
      "y": "-V4dS4UaLMgP_4fY4j8ir7cl1TXlFdAgcx55o7TkcSA"
    }
  }
}
```

See (#wallet-attestation-example) for an example of a signed Wallet Attestation JWT.

To use the Wallet Attestation towards the Authorization Server, the Wallet MUST generate a proof of possession according to Section 5.2 "Client Attestation PoP JWT" of Attestation-Based Client Authentication.

The `sub` claim of the Wallet Attestation JWT is the `client_id` of the Wallet instance. For privacy reasons, this value is the same across Wallet instances of that Wallet Provider, see (#walletattestation-sub) for more details.

# Proof Types {#proof-types}

A proof type communicates a proof of cryptographic key material used for binding a Credential in the Credential Request.

This specification defines the following proof types:

* `jwt`: A JWT [@!RFC7519] is used for proof of possession. When a `proofs` object is using a `jwt` proof type, it MUST include a `jwt` parameter with its value being a non-empty array of JWTs, where each JWT is formed as defined in (#jwt-proof-type).
* `di_vp`: A W3C Verifiable Presentation object signed using the Data Integrity Proof [@VC_Data_Integrity] as defined in [@VC_DATA_2.0] or [@VC_DATA] is used for proof of possession. When a `proofs` object is using a `di_vp` proof type, it MUST include an `di_vp` parameter with its value being a non-empty array of W3C Verifiable Presentations as defined by [@VC_DATA_2.0] or [@VC_DATA], where each of these W3C Verifiable Presentation is formed as defined in (#di-vp-proof-type).
* `attestation`:  A JWT [@!RFC7519] representing a key attestation without using a proof of possession of the cryptographic key material that is being attested. When a `proofs` object is using an `attestation` proof type, the object MUST include an `attestation` parameter with its value being an array that contains exactly one JWT that is formed as defined in (#keyattestation-jwt).

There are two ways to convey key attestations (as defined in (#keyattestation)) of the cryptographic key material during Credential issuance:

- The Wallet uses the `jwt` proof type in the Credential Request to create a proof of possession for one of the attested keys and adds the key attestation in the JOSE header.
- The Wallet uses the `attestation` proof type in the Credential Request to provide a key attestation without a proof of possession of any of the keys.

Depending on the Wallet's implementation, the `attestation` may avoid unnecessary End-User interaction during Credential issuance, as the key(s) to which the Credential(s) will be bound does not necessarily need to perform signature operations, and one key attestation can be used to attest multiple keys.

Additional proof types MAY be defined and used.

## `jwt` Proof Type {#jwt-proof-type}

The JWT MUST contain the following elements:

* in the JOSE header,
  * `alg`: REQUIRED. A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry [@IANA.JOSE]. It MUST NOT be `none` or an identifier for a symmetric algorithm (MAC).
  * `typ`: REQUIRED. MUST be `openid4vci-proof+jwt`, which explicitly types the key proof JWT as recommended in Section 3.11 of [@!RFC8725].
  * `kid`: OPTIONAL. JOSE Header containing the key ID. If the Credential is to be bound to a DID, the `kid` refers to a DID URL which identifies a particular key in the DID Document that the Credential is to be bound to. It MUST NOT be present if `jwk` or `x5c` is present.
  * `jwk`: OPTIONAL. JOSE Header containing the key material the new Credential is to be bound to. It MUST NOT be present if `kid` or `x5c` is present.
  * `x5c`: OPTIONAL. JOSE Header containing at least one certificate where the first certificate contains the key that the Credential is to be bound to, additional certificates may also be present. It MUST NOT be present if `kid` or `jwk` is present.
  * `key_attestation`: OPTIONAL. JOSE Header containing a key attestation as described in (#keyattestation). If the Credential Issuer provided a `c_nonce`, the `nonce` claim in the key attestation MUST be set to a server-provided `c_nonce`.
  * `trust_chain`: OPTIONAL. JOSE Header containing an [@!OpenID.Federation] Trust Chain. This element MAY be used to convey key attestation, metadata, metadata policies, federation Trust Marks and any other information related to a specific federation, if available in the chain. When used for signature verification, the header parameter `kid` MUST be present.

* in the JWT body,
  * `iss`: OPTIONAL (string). The value of this claim MUST be the `client_id` of the Client making the Credential request. This claim MUST be omitted if the access token authorizing the issuance call was obtained from a Pre-Authorized Code Flow through anonymous access to the token endpoint.
  * `aud`: REQUIRED (string). The value of this claim MUST be the Credential Issuer Identifier.
  * `iat`: REQUIRED (number). The value of this claim MUST be the time at which the key proof was issued using the syntax defined in [@!RFC7519].
  * `nonce`: OPTIONAL (string). The value type of this claim MUST be a string, where the value is a server-provided `c_nonce`. It MUST be present when the issuer has a Nonce Endpoint as defined in (#nonce-endpoint).

The Credential Issuer MUST validate that the JWT used as a proof is actually signed by a key identified in the JOSE Header through either `kid`, `jwk` or `x5c` element.

The Credential Issuer SHOULD issue a Credential for each cryptographic public key specified in the `attested_keys` claim within the `key_attestation` parameter.

Cryptographic algorithm identifiers used in the `proof_signing_alg_values_supported` Credential Issuer metadata parameter for this proof type are case sensitive strings and SHOULD be one of the Algorithm Names defined in [@IANA.JOSE].
If Credential Issuer metadata is provided, the `alg` JWT header of the key proof, and if present, the `alg` JOSE headers of both `key_attestation` and `trust_chain`, MUST match one of the values listed in the `proof_signing_alg_values_supported` metadata parameter.

Below is a non-normative example of a `proofs` parameter (with line breaks within values for display purposes only):

```json
{
  "jwt": [
    "eyJ0eXAiOiJvcGVuaWQ0dmNpLXByb29mK2p3dCIsImFsZyI6IkVTMjU2Iiwiand
    rIjp7Imt0eSI6IkVDIiwiY3J2IjoiUC0yNTYiLCJ4IjoiblVXQW9BdjNYWml0aDh
    FN2kxOU9kYXhPTFlGT3dNLVoyRXVNMDJUaXJUNCIsInkiOiJIc2tIVThCalVpMVU
    5WHFpN1N3bWo4Z3dBS18weGtjRGpFV183MVNvc0VZIn19.eyJhdWQiOiJodHRwcz
    ovL2NyZWRlbnRpYWwtaXNzdWVyLmV4YW1wbGUuY29tIiwiaWF0IjoxNzAxOTYwND
    Q0LCJub25jZSI6IkxhclJHU2JtVVBZdFJZTzZCUTR5bjgifQ.-a3EDsxClUB4O3L
    eDD5DVGEnNMT01FCQW4P6-2-BNBqc_Zxf0Qw4CWayLEpqkAomlkLb9zioZoipdP-
    jvh1WlA"
  ]
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

Here is another example of a JWT not only proving possession of a private key but also providing key attestation data for that key:

```json
{
  "typ": "openid4vci-proof+jwt",
  "alg": "ES256",
  "kid": "0",
  "key_attestation": <key attestation in JWT format>
}.
{
  "iss": "s6BhdRkqt3",
  "aud": "https://server.example.com",
  "iat": 1659145924,
  "nonce": "tZignsnFbp"
}
```

## `di_vp` Proof Type {#di-vp-proof-type}

When a W3C Verifiable Presentation as defined by [@VC_DATA_2.0] or [@VC_DATA] secured using Data Integrity [@VC_Data_Integrity] is used as key proof, it MUST contain at least the following properties, in addition to any other properties required by [@VC_DATA_2.0] or [@VC_DATA]:

* `holder`: OPTIONAL. If present, it MUST be equivalent to the controller identifier part (e.g., DID) of the `verificationMethod` value identified by the `proof.verificationMethod` property.
* `proof`: REQUIRED. The `proof` property of the W3C Verifiable Presentation MUST be a Data Integrity Proof, as defined in [@VC_Data_Integrity], and its properties MUST conform with the following rules, in addition to those specified in [@VC_Data_Integrity]:  
  * `cryptosuite`: REQUIRED. If Credential Issuer metadata is provided, the value MUST match one of the entries in the `proof_signing_alg_values_supported` metadata parameter.
  * `proofPurpose`: REQUIRED. MUST be set to `authentication`.
  * `domain`: REQUIRED. MUST be set to the Credential Issuer Identifier.  
  * `challenge`: REQUIRED when the Credential Issuer has provided a `c_nonce`. It MUST NOT be used otherwise. String, where the value is a server-provided `c_nonce`. It MUST be present when the issuer has a Nonce Endpoint as defined in (#nonce-endpoint).

The Credential Issuer MUST validate that the W3C Verifiable Presentation used as a proof is actually signed with a key in the possession of the Holder.

Additional properties in the W3C Verifiable Presentation and Data Integrity Proof MUST be ignored if not understood.
Cryptographic algorithm identifiers used in the `proof_signing_alg_values_supported` Credential Issuer metadata parameter for this proof type are case sensitive strings and SHOULD be one of those defined in, or referenced by, [@VC_Data_Integrity].

Below is a non-normative example of a `proofs` parameter (with line breaks within values for display purposes only):

```json
{
  "di_vp": [
    {
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
          "verificationMethod": "did:key:z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mqh5
           XPvLsrPsro#z6MkvrFpBNCoYewiaeBLgjUDvLxUtnK5R6mqh5XPvLsrPsro",
          "created": "2023-03-01T14:56:29.280619Z",
          "challenge": "82d4cb36-11f6-4273-b9c6-df1ac0ff17e9",
          "domain": "did:web:audience.company.com",
          "proofValue": "z5hrbHzZiqXHNpLq6i7zePEUcUzEbZKmWfNQzXcUXUrqF7bykQ7ACi
           WFyZdT2HcptF1zd1t7NhfQSdqrbPEjZceg7"
        }
      ]
    }
  ]
}

```

## `attestation` Proof Type {#attestation-proof-type}

A key attestation in JWT format as defined in (#keyattestation-jwt).

If the Credential Issuer has a Nonce Endpoint (as defined in (#nonce-endpoint)), the `c_nonce` value provided by the Credential Issuer MUST be provided in the key attestation's `nonce` parameter.

Cryptographic algorithm identifiers used in the `proof_signing_alg_values_supported` Credential Issuer metadata parameter for this proof type are case sensitive strings and SHOULD be one of those defined in [@IANA.JOSE].

Below is a non-normative example of a `proofs` parameter (with line breaks within values for display purposes only):

```json
{
  "attestation": [
    "<key attestation in JWT format>"
  ]
}
```

The Credential Issuer SHOULD issue a Credential for each cryptographic public key specified in the `attested_keys` claim within the `key_attestation` parameter.

If Credential Issuer metadata is provided, the value of the `alg` JWT header of the key attestation MUST match one of the entries in the `proof_signing_alg_values_supported` metadata parameter.

## Verifying Proof {#verifying-key-proof}

To validate a key proof, the Credential Issuer MUST ensure that:

- all required claims for that proof type are contained as defined in (#proof-types),
- the key proof is explicitly typed using header parameters as defined for that proof type,
- the header parameter indicates a registered asymmetric digital signature algorithm, `alg` parameter value is not `none`, is supported by the application, and is acceptable per local policy,
- the signature on the key proof verifies with the public key contained in the header parameter,
- the header parameter does not contain a private key,
- if the server has a Nonce Endpoint, the nonce in the key proof matches the server-provided `c_nonce` value,
- the creation time of the JWT, as determined by either the issuance time, or a server managed timestamp via the nonce claim, is within an acceptable window (see (#key-proof-replay)).

These checks may be performed in any order.

# IANA Considerations

## OAuth URI Registry

This specification registers the following URN
in the IANA "OAuth URI" registry [@IANA.OAuth.Parameters]
established by [@!RFC6755].

### urn:ietf:params:oauth:grant-type:pre-authorized_code

* URN: `urn:ietf:params:oauth:grant-type:pre-authorized_code`
* Common Name: Pre-Authorized Code
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential-offer-parameters) of this specification

## OAuth Parameters Registry

This specification registers the following OAuth parameter
in the IANA "OAuth Parameters" registry [@IANA.OAuth.Parameters]
established by [@!RFC6749].

### interaction_types_supported

* Name: `interaction_types_supported`
* Parameter Usage Location: authorization request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#interactive-authorization-request) of this specification

### issuer_state

* Name: `issuer_state`
* Parameter Usage Location: authorization request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential-authz-request) of this specification

### pre-authorized_code

* Name: `pre-authorized_code`
* Parameter Usage Location: token request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-request) of this specification

### tx_code

* Name: `tx_code`
* Parameter Usage Location: token request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-request) of this specification

## OAuth Authorization Server Metadata Registry

This specification registers the following authorization server metadata parameter
in the IANA "OAuth Authorization Server Metadata" registry [@IANA.OAuth.Parameters]
established by [@!RFC8414].

### pre-authorized_grant_anonymous_access_supported

* Metadata Name: `pre-authorized_grant_anonymous_access_supported`
* Metadata Description: Boolean indicating whether Credential Issuer accepts Token Request with Pre-Authorized Code but without `client_id`
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#as-metadata) of this specification

### interactive_authorization_endpoint

* Metadata Name: `interactive_authorization_endpoint`
* Metadata Description: URL of the Authorization Server's Interactive Authorization Endpoint. This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Authorization Server does not support the Interactive Authorization Endpoint.
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#as-metadata) of this specification

## OAuth Dynamic Client Registration Metadata Registry

This specification registers the following client metadata parameter
in the IANA "OAuth Dynamic Client Registration Metadata" registry [@IANA.OAuth.Parameters]
established by [@!RFC7591].

### credential_offer_endpoint

* Client Metadata Name: `credential_offer_endpoint`
* Client Metadata Description: Credential Offer Endpoint
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#client-metadata) of this specification

## OAuth Extensions Error Registry

This specification registers the following errors in the IANA "OAuth Extensions Error" registry [@IANA.OAuth.Parameters] established by [@!RFC6755].

### missing_interaction_type

* Error name: `missing_interaction_type`
* Error usage location: Interactive Authorization Error Response
* Related protocol extension: OpenID for Verifiable Credential Issuance
* Change controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Specification document: (#iae-error-response) of this specification

## Well-Known URI Registry

This specification registers the following well-known URI
in the IANA "Well-Known URI" registry [@IANA.Well-Known.URIs]
established by [@!RFC8615].

### .well-known/openid-credential-issuer

* URI suffix: `openid-credential-issuer`
* Change controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Specification document: (#credential-issuer-wellknown) of this specification
* Related information: (none)

## Media Types Registry

This specification registers the following media type [@RFC2046]
in the IANA "Media Types" registry [@IANA.MediaTypes]
in the manner described in [@RFC6838].

### application/openid4vci-proof+jwt

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

### application/key-attestation+jwt

* Type name: `application`
* Subtype name: `key-attestation+jwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Uses JWS Compact Serialization, as specified in [@!RFC7515].
* Security considerations: See the Security Considerations in [@!RFC7519].
* Interoperability considerations: n/a
* Published specification: (#keyattestation-jwt) of this specification
* Applications that use this media type: Applications that use the key attestation format defined in this specification
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

### application/openidvci-issuer-metadata+jwt

* Type name: `application`
* Subtype name: `openidvci-issuer-metadata+jwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Uses JWS Compact Serialization, as specified in [@!RFC7515].
* Security considerations: See the Security Considerations in [@!RFC7519].
* Interoperability considerations: n/a
* Published specification: (#credential-issuer-signed-metadata) of this specification
* Applications that use this media type: Applications that use signed metadata format defined in this specification
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

## Uniform Resource Identifier (URI) Schemes Registry

This specification registers the following URI scheme
in the IANA "Uniform Resource Identifier (URI) Schemes" registry [@IANA.URI.Schemes].

### openid-credential-offer

* URI Scheme: openid-credential-offer
* Description: Custom scheme used for credential offers
* Status: Permanent
* Well-Known URI Support: -
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#client-metadata-retrieval) of this specification

# Use Cases

This is a non-exhaustive list of sample use cases.

## Credential Offer - Same-Device {#use-case-1}

While browsing the university's home page, the End-User finds a link "request your digital diploma". The End-User clicks on this link and is redirected to a digital Wallet. The Wallet notifies the End-User that a Credential Issuer offered to issue a diploma Credential. The End-User confirms this inquiry and is taken to the university's Credential issuance service's End-User experience. Upon successful authentication at the university and consent to the issuance of a digital diploma, the End-User is redirected back to the Wallet. Here, the End-User can verify the successful creation of the digital diploma.

## Credential Offer - Cross-Device (with Information Pre-Submitted by the End-User) {#use-case-2}

The End-User is starting a job with a new employer. The employer requests the End-User to upload specific documents to the employee portal. After a few days, the End-User receives an email from the employer indicating that the employee Credential is ready to be claimed and provides instructions to scan a presented QR code for its retrieval. The End-User scans the QR code with a smartphone, which opens the Wallet. Meanwhile, the End-User has received a text message with a Transaction Code to the smartphone. After entering the Transaction Code in the Wallet for security reasons, the End-User approves the Credential issuance and receives the Credential in the Wallet.

## Credential Offer - Cross-Device & Deferred {#use-case-3}

The End-User intends to acquire a digital criminal record. This involves a visit to the local administration's office to request the official criminal record be issued as a digital Credential. After presenting an ID document, the End-User is prompted to scan a QR code using the Wallet and is informed that the issuance of the Credential will require some time, due to necessary background checks by the authority.

While using the Wallet, the End-User notices an indication that the issuance of the digital record is in progress. After a few days, the End-User receives a notification from the Wallet indicating that the requested Credential was successfully issued. Upon opening the Wallet, the End-User is queried about the download of the Credential. After confirmation, the Wallet fetches and saves the new Credential.

## Wallet-Initiated Issuance during Presentation {#use-case-4}

An End-User comes across a Verifier app that is requesting the End-User to present a Credential, e.g., a driving license. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet selects a Credential Issuer capable of issuing the missing Credential and, upon End-User consent, sends the End-User to the Credential Issuer's End-User experience (website or app). Once authenticated and consent is provided for the issuance of the Credential into the Wallet, the End-User is redirected back to the Wallet. The Wallet informs the End-User that Credential was successfully issued into the Wallet and is ready to be presented to the Verifier app that originally requested presentation of that Credential.

## Wallet-Initiated Issuance during Presentation (Requires Presentation of Additional Credentials During Issuance) {#use-case-5}

An End-User comes across a Verifier app that is requesting the End-User to present a Credential, e.g., a university diploma. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet then offers the End-User a list of Credential Issuers, which might be based on a Credential Issuer list curated by the Wallet provider. The End-User selects the university of graduation and is subsequently redirected to the corresponding university's website or app.

The End-User logs into the university, which identifies that the corresponding End-User account is not yet verified. Among various identification options, the End-User opts to present a Credential from the Wallet. The End-User is redirected back to the Wallet to consent to present the requested Credential(s) to the university. Following this, the End-User is redirected back to the university End-User experience. Based on the presented Credential, the university finalizes the End-User verification, retrieves data about the End-User from its database, and proposes to issue a diploma as a Verifiable Credential.

Upon providing consent, the End-User is sent back to the Wallet. The Wallet informs the End-User that the Credential was successfully issued into the Wallet and is ready to be presented to the Verifier app that originally requested presentation of that Credential.

## Wallet-Initiated Issuance after Installation {#use-case-6}

The End-User installs a new Wallet and opens it. The Wallet offers the End-User a selection of Credentials that the End-User may obtain from a Credential Issuer, e.g., a national identity Credential, a mobile driving license, or a public transport ticket. The corresponding Credential Issuers (and their URLs) are pre-configured by the Wallet or follow some discovery processes that are out of scope for this specification. By clicking on one of these options corresponding to the Credentials available for issuance, the issuance process starts using a flow supported by the Credential Issuer (Pre-Authorized Code flow or Authorization Code flow).

Wallet Providers may also provide a market place where Issuers can register to be found for Wallet-initiated flows.

# Additional Examples {#additional-examples}

## Credential Issuer Metadata {#additional-issuer-metadata-examples}

This section contains additional examples for Credential issuer Metadata

The following is a non-normative example of Credential Issuer metadata of a Credential in the IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format in signed form (JWT):

<{{examples/credential_issuer_metadata_sd_jwt_jwt.txt}}

The following is a non-normative example of Credential Issuer metadata of a Credential in the IETF SD-JWT VC [@!I-D.ietf-oauth-sd-jwt-vc] format with optional parameters present:

<{{examples/credential_issuer_metadata_sd_jwt_long.json}}

## Wallet Attestation JWT {#wallet-attestation-example}

The following is a non-normative example of a signed Wallet Attestation:

<{{examples/wallet_attestation.jwt.txt}}

# Acknowledgements {#Acknowledgements}

We would like to thank Patrick Amrein, Masayoshi Arakawa, Richard Barnes, Paul Bastian, Vittorio Bertocci, Christian Bormann, John Bradley, Brian Campbell, Lee Campbell, Tim Cappalli, Lenah Chacha, David Chadwick, Stefan Charsley, Gabe Cohen, Thomas Darimont, Andrii Deinega, Giuseppe De Marco, Rajvardhan Deshmukh, Mark Dobrinic, Daniel Fett, Pedro Felix, George Fletcher, Christian Fries, Timo Glasta, Mark Haine, Martijn Haring, Fabian Hauck, Roland Hedberg, Joseph Heenan, Bjorn Hjelm, Alen Horvat, Andrew Hughes, Jacob Ideskog, Lukasz Jaromin, Edmund Jay, Michael B. Jones, Tom Jones, Judith Kahrer, Micha Kraus, Takahiko Kawasaki, Niels Klomp, Ronald Koenig, Micha Kraus, Markus Kreusch, Philipp Lehwalder, Adam Lemmon, Dave Longley, Hicham Lozi, David Luna, Daniel McGrogan, Jeremie Miller, Mirko Mollik, Kenichi Nakamura, Andres Olave, Gareth Oliver, Nemanja Patrnogic, Dima Postnikov, Rolson Quadras, Helen Quinn, Sami Rosendahl, Babis Routis, Nat Sakimura, Sudesh Shetty, Peter Sorotokin, Oliver Terbu, Dimitri James Tsiflitzis, Mike Varley, Niels van Dijk, Arjen van Veen, Jan Vereecken, David Waite, Jacob Ward, David Zeuthen for their valuable feedback and contributions to this specification.

# Notices

Copyright (c) 2025 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft, Final Specification, or Final Specification Incorporating Errata Corrections solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts, Final Specifications, and Final Specification Incorporating Errata Corrections based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy (found at openid.net) requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. OpenID invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -01

   * Initial draft created with same text as 1.0 Final
   * Add back Interactive Authorization Endpoint text that was removed from the 1.0 draft
   * move IAE binding to dedicated format-specific sections
   * rename `iar:` prefix in `iae:` in IAE flow
   * rename `iar-post` response mode in `iae_post` in IAE flow
   * use derived origin for `expected_origins` in IAE flow
   * add require_interactive_authorization_request to AS metadata
   * add interactive_authorization_endpoint to AS metadata section
