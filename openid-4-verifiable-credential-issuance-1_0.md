%%%
title = "OpenID for Verifiable Credential Issuance - Editor's draft"
abbrev = "openid-4-verifiable-credential-issuance"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-4-verifiable-credential-issuance-1_0-13"
status = "standard"

[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="sprind.org"
    [author.address]
    email = "torsten@lodderstedt.net"

[[author]]
initials="K."
surname="Yasuda"
fullname="Kristina Yasuda"
organization="Microsoft"
    [author.address]
    email = "kristina.yasuda@microsoft.com"

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

This specification defines an API that is used to issue Verifiable Credentials. W3C formats [@VC_DATA] as well as other Credential formats, like [@ISO.18013-5], are supported. 

Verifiable Credentials are very similar to identity assertions, like ID Tokens in OpenID Connect [@OpenID.Core], in that they allow a Credential Issuer to assert End-User claims. A Verifiable Credential follows a pre-defined schema (the Credential type) and MAY be bound to a certain holder, e.g., through Cryptographic Holder Binding. Verifiable Credentials can be securely presented for the End-User to the RP, without involvement of the Credential Issuer.

Access to this API is authorized using OAuth 2.0 [@!RFC6749], i.e., the Wallet uses OAuth 2.0 to obtain authorization to receive Verifiable Credentials. This way the issuance process can benefit from the proven security, simplicity, and flexibility of OAuth 2.0 and existing OAuth 2.0 deployments and OpenID Connect OPs (see [@OpenID.Core]) can be extended to become Credential Issuers.

## Requirements Notation and Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [@!RFC2119].

# Terminology

This specification uses the terms "Access Token", "Authorization Endpoint", "Authorization Request", "Authorization Response", "Authorization Code Grant", "Authorization Server", "Client", "Client Authentication", "Client Identifier", "Grant Type", "Refresh Token", "Token Endpoint", Token Request" and "Token Response" defined by OAuth 2.0 [@!RFC6749], the terms "End-User", "Entity", and "Request Object" as defined by OpenID Connect Core [@!OpenID.Core], the term "JSON Web Token (JWT)" defined by JSON Web Token (JWT) [@!RFC7519], the term "JOSE Header" and "Base64url Encoding" defined by JSON Web Signature (JWS) [@!RFC7515].

This specification also defines the following terms. In the case where a term has a definition that differs, the definition below is authoritative for this specification.

Credential:
:  A set of one or more claims about a subject made by a Credential Issuer. Note that this definition of a term "credential" in this specification is different from that in [@!OpenID.Core] and [@!RFC6749].

Verifiable Credential (VC):
:  An Issuer-signed Credential whose integrity can be cryptographically verified. It can be of any format used in the Issuer-Holder-Verifier Model, including, but not limited to those defined in [@VC_DATA] and [@ISO.18013-5].

W3C Verifiable Credential:
:  A Verifiable Credential compliant to the [@VC_DATA] specification.

Presentation:
: Data that is presented to a specific verifier, derived from one or more Verifiable Credentials that can be from the same or different Credential Issuers.

Verifiable Presentation (VP):
:  A Holder-signed Credential whose integrity can be cryptographically verified to provide Cryptographic Holder Binding. It can be of any format used in the Issuer-Holder-Verifier Model, including, but not limited to those defined in [@VC_DATA] and [@ISO.18013-5].

W3C Verifiable Presentation:
:  A Verifiable Presentations compliant to the [@VC_DATA] specification.

Credential Issuer:
:  An entity that issues Verifiable Credentials. Also called Issuer. In the context of this specification, the Credential Issuer acts as an OAuth 2.0 Authorization Server (see [@!RFC6749]).

Holder:
:  An entity that receives Verifiable Credentials and has control over them to present them to the Verifiers as Verifiable Presentations.

Verifier:
:  An entity that requests, receives, and validates Verifiable Presentations. During presentation of Credentials, Verifier acts as an OAuth 2.0 Client towards the Wallet that is acting as an OAuth 2.0 Authorization Server. The Verifier is a specific case of an OAuth 2.0 Client, just like Relying Party (RP) in [@OpenID.Core].

Issuer-Holder-Verifier Model:
:  A model for exchanging claims, where claims are issued in the form of Verifiable Credentials independent of the process of presenting them as Verifiable Presentations to the Verifiers. An issued Verifiable Credential can be (but is not necessarily) used multiple times.

Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential.

Cryptographic Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proving control over the same private key during the issuance and presentation. Mechanism might depend on the Credential Format. For example, in `jwt_vc_json` Credential Format, a VC with Cryptographic Holder Binding contains a public key or a reference to a public key that corresponds to the private key controlled by the Holder.

Claim-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proofing certain claims, e.g., name and date of birth, for example by presenting another Verifiable Credential. Claim-based Holder Binding allows long-term, cross-device use of a Credential as it does not depend on cryptographic key material stored on a certain device. One example of such a Verifiable Credential could be a Diploma.

Biometrics-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by demonstrating a certain biometric trait, such as fingerprint or face. One example of a Verifiable Credential with Biometrics-based Holder Binding is a mobile driving license [@ISO.18013-5], which contains a portrait of the holder.

Wallet:
:  An entity used by the Holder to request, receive, store, present, and manage Verifiable Credentials and key material. There is no single deployment model of a Wallet: Verifiable Credentials and keys can both be stored and managed locally, or by using a remote self-hosted service, or a remote third-party service. In the context of this specification, the Wallet acts as an OAuth 2.0 Client since the Credential Issuer is the OAuth 2.0 Resource Server and, in certain cases, it may also implement an OAuth 2.0 Authorization Server.

Deferred Credential Issuance:
:  Issuance of Credentials not directly in the response to a Credential issuance request but following a period of time that can be used to perform certain offline business processes.

# Overview

## Credential Issuer

This specification defines an API for Credential issuance provided by a Credential Issuer. The API is comprised of the following endpoints:

* A mandatory Credential Endpoint from which Credentials can be issued (see (#credential-endpoint)).
* An optional Batch Credential Endpoint from which multiple Credentials can be issued in one request (see (#batch-credential-endpoint)).
* An optional Deferred Credential Endpoint to allow for the deferred delivery of Credentials (see (#deferred-credential-issuance)).
* An optional mechanism for the Credential Issuer to make a Credential Offer to the Wallet to encourage the Wallet to start the issuance flow (see (#credential_offer_endpoint)).
* An optional mechanism for the Credential Issuer to receive from the Wallet notification(s) of the status of the Credential(s) that have been issued.
* A mechanism for the Credential Issuer to publish metadata about the Credentials it is capable of issuing (see (#credential_issuer_metadata)).

Both the Credential and the Batch Credential Endpoints have the (optional) ability to bind an issued Credential to certain cryptographic key material. Both requests therefore enable conveying proof of possession for the key material. Multiple key proof types are supported.

## OAuth 2.0

Every Credential Issuer utilizes an OAuth 2.0 [@!RFC6749] Authorization Server to authorize access. The same OAuth 2.0 Authorization Server can protect one or more Credential Issuers. Wallets determine the Credential Issuer's Authorization Server using the Credential Issuer's metadata (see (#credential_issuer_metadata)).

All OAuth 2.0 Grant Types and extensions mechanisms can be used in conjunction with the Credential issuance API. Aspects not defined in this specification are expected to follow [@!RFC6749]. 

Existing OAuth 2.0 mechanisms are extended as following:

* A new Grant Type "Pre-Authorized Code" is defined to facilitate flows where the preparation of the Credential issuance is conducted before the actual OAuth flow starts (see (#pre-authz-code-flow)).
* A new authorization details [@!RFC9396] type `openid_credential` is defined to convey the details about the Credentials (including formats and types) the Wallet wants to obtain (see (#authorization-details)).
* New token response error codes `authorization_pending` and `slow_down` are added to allow for deferred authorization of Credential issuance. These error codes are supported for the Pre-Authorized Code grant type.
* Client metadata is used to convey Wallet's metadata. A new metadata parameter `credential_offer_endpoint` is added to allow a Wallet (acting as OAuth 2.0 client) to publish its Credential Offer Endpoint (see (#client_metadata)).
* Authorization Endpoint: An additional parameter `issuer_state` is added to convey state in the context of processing an issuer-initiated Credential Offer (see (#credential-authz-request)). Additional parameters `wallet_issuer` and `user_hint` are added to enable the Credential Issuer to request Verifiable Presentations from the calling Wallet during Authorization Request processing.
* Token Endpoint: optional response parameters `c_nonce` and `c_nonce_expires_in` are added to the Token Endpoint, Credential Endpoint and Batch Credential Endpoint to provide the Client with a nonce to be used for proof of possession of key material in a subsequent request to the Credential Endpoint (see (#token-response)).

## Core Concepts

The Wallet sends one Credential Request per individual Credential to the Credential Endpoint. The Wallet MAY use the same Access Token to send multiple Credential Requests to request issuance of the following:

* multiple Credentials of different types/doctypes bound to the same proof, or
* multiple Credentials of the same type/doctype bound to different proofs, or
* multiple Credentials of the different types/doctypes bound to different proofs.

Note: "type" and "doctype" are terms defined by an individual Credential format. For details, see (#format_profiles).

The Wallet MAY send one Batch Credential Request to the Batch Credential Endpoint to request the following in the Batch Credential Response:

* multiple Credentials of different types/doctypes bound to the same proof, or 
* multiple Credentials of the same type/doctype bound to different proofs, or
* multiple Credentials of different types/doctypes bound to different proofs.

In the course of the authorization process, the Credential Issuer MAY also request Credential presentation as a means to authenticate or identify the End-User during the issuance flow, as illustrated in (#use-case-5).

At its core, this specification is Credential format agnostic and allows implementers to leverage specific capabilities of Credential formats of their choice. Multiple Credential formats can be used within the same transaction. 

The specification achieves this by defining the following:

* Extension points to add Credential format specific parameters or claims in the Credential Issuer metadata, Credential Offer, Authorization Request, Credential Request and Batch Credential Request,
* Credential format identifiers to identify Credential format specific set of parameters and claims to be applied at each extension point. This set of Credential format specific set of parameters and claims is referred to as a "Credential Format Profile" in this specification.

This specification defines Credential Format Profiles for W3C Verifiable Credentials as defined in [@VC_DATA] and ISO/IEC 18013-5 mDL as defined in [@ISO.18013-5] in (#format_profiles) that contain Credential Format specific parameters to be included at each extension point defined in this specification. Other specifications or deployments can define their own Credential Format Profiles using the above-mentioned extension points.

The issuance can have multiple characteristics, which can be combined depending on the use cases:

* Authorization Code Flow or Pre-Authorized Code Flow: The Credential Issuer can obtain End-User information to turn into a Verifiable Credential using End-User authentication and consent at the Credential Issuer's Authorization Endpoint (Authorization Code Flow) or using out-of-band mechanisms outside of the issuance flow (Pre-Authorized Code Flow).
* Wallet initiated or Issuer initiated: The request from the Wallet can be sent to the Credential Issuer without any gesture from the Credential Issuer (Wallet Initiated) or following the communication from the Credential Issuer (Issuer Initiated).
* Same-device or Cross-device Credential Offer: The End-User may receive the Credential Offer from the Credential Issuer either on the same device as the device the Wallet resides on, or through any other means, such as another device or physical mail, so that the Credential Offer can be communicated to the Wallet.
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
+----------+        +-----------+                                    +-------------------+
| End-User |        |   Wallet  |                                    | Credential Issuer |
+----------+        +-----------+                                    +-------------------+
    |                     |                                                    |
    | (1a) End-User       |                                                    |
    |  selects Credential |  (1b) Credential Offer (credential type)           |
    |-------------------->|<---------------------------------------------------|
    |                     |                                                    |
    |                     |  (2) Obtains Issuer's Credential Issuer metadata   |
    |                     |<-------------------------------------------------->|
    |                     |                                                    |
    |                     |  (3) Authorization Request                         |
    |                     |      (type(s) of Credentials to be issued)         |
    |                     |--------------------------------------------------->|
    |                     |                                                    |
    |           End-User Authentication / Consent                              |
    |                     |                                                    |
    |                     |  (4)   Authorization Response (code)               |
    |                     |<---------------------------------------------------|
    |                     |                                                    |
    |                     |  (5) Token Request (code)                          |
    |                     |--------------------------------------------------->|
    |                     |      Token Response (Access Token)                 |
    |                     |<---------------------------------------------------|
    |                     |                                                    |
    |                     |  (6) Credential Request (Access Token, proof(s))   |
    |                     |--------------------------------------------------->|
    |                     |      Credential Response                           |
    |                     |      (Credential(s) OR Transaction ID)             |
    |                     |<---------------------------------------------------|
~~~
!---
Figure: Issuance using Authorization Code Flow 

(1a) The Wallet-initiated flow begins as the End-User requests a Credential via the Wallet from the Credential Issuer. The End-User either selects a Credential from a pre-configured list of Credentials ready to be issued, or, alternatively, the Wallet gives guidance to the End-User to select a Credential from a Credential Issuer based on the information it received in the presentation request from a Verifier.

(1b) The Issuer-initiated flow begins as the Credential Issuer generates a Credential Offer for certain Credential(s) that it communicates to the Wallet, for example, as a QR code or as a URI. The Credential Offer contains the Credential Issuer's URL and the information about the Credential(s) being offered. This step is defined in (#credential_offer).

(2) The Wallet uses the Credential Issuer's URL to fetch the Credential Issuer metadata as described in (#credential_issuer_metadata). The Wallet needs the metadata to learn the Credential types and formats that the Credential Issuer supports, and to determine the Authorization Endpoint (OAuth 2.0 Authorization Server) as well as Credential Endpoint required to start the request. This specification enables deployments where the Credential Endpoint and the Authorization Endpoint are provided by different entities. Please note that in this example the Credential Issuer and OAuth 2.0 Authorization Server correspond to the same entity.

(3) The Wallet sends an Authorization Request to the Authorization Endpoint. The Authorization Endpoint processes the Authorization Request, which typically includes the End-User authentication and the gathering of the End-User consent. Note: The Authorization Request may be sent as a Pushed Authorization Request.

Note: Steps (3) and (4) happen in the front channel, by redirecting the End-User via the User Agent. Those steps are defined in (#authorization_endpoint). The Authorization Server and the User Agent may exchange any further messages between the steps if required by the Authorization Server to authenticate the End-User.

(4) The Authorization Endpoint returns the Authorization Response with the Authorization Code upon successfully processing the Authorization Request.

(5) The Wallet sends a Token Request to the Token Endpoint with the Authorization Code obtained in step (4). The Token Endpoint returns an Access Token in the Token Response upon successfully validating the Authorization Code. This step happens in the back-channel communication (direct communication between two systems using HTTP requests and responses without using redirects through an intermediary such as a browser). This step is defined in (#token_endpoint).

(6) The Wallet sends a Credential Request to the Credential Issuer's Credential Endpoint with the Access Token and (optionally) the proof of possession of the private key of a key pair to which the Credential Issuer should bind the issued Credential to. Upon successfully validating Access Token and proof, the Credential Issuer returns a Credential in the Credential Response. This step is defined in (#credential-endpoint).

If the Credential Issuer requires more time to issue a Credential, the Credential Issuer may return a Transaction ID and a time interval in the Credential Response. The Wallet may send a Deferred Credential Request with the Transaction ID to obtain a Credential after the specified time interval has passed, as defined in (#deferred-credential-issuance).

If the Credential Issuer wants to issue multiple Credentials in one response, the Credential Issuer may support the Batch Credential Endpoint. In this case the Wallet may send a Batch Credential Request to the Batch Credential Endpoint, as defined in (#batch-credential-endpoint).

Note: This flow is based on OAuth 2.0 and the Authorization Code Grant type, but this specification can be used with other OAuth 2.0 grant types as well.

## Pre-Authorized Code Flow {#pre-authz-code-flow}

Figure 2 is a diagram of a Credential issuance using the Pre-Authorized Code Flow. In this flow, before initiating the flow with the Wallet, the Credential Issuer first conducts the steps required to prepare the Credential issuance, e.g., End-User authentication and authorization. Consequently, the Pre-Authorized Code is sent by the Credential Issuer to the Wallet. This flow does not use the Authorization Endpoint, and the Wallet exchanges the Pre-Authorized Code for the Access Token directly at the Token Endpoint. Access Token is then used to request Credential issuance at the Credential Endpoint. See (#use-case-2) for the description of a use case.

How the End-User provides information required for the issuance of a requested Credential to the Credential Issuer and the business processes conducted by the Credential Issuer to prepare a Credential are out of scope of this specification.

This flow uses the newly defined OAuth 2.0 Grant Type "urn:ietf:params:oauth:grant-type:pre-authorized_code".

The following diagram is based on the Credential Issuer initiated flow, as illustrated in a use case in (#use-case-2). Please note that it does not illustrate all the optional features outlined in this specification.

!---
~~~ ascii-art
+--------------+   +-----------+                                    +-------------------+
|   End-User   |   |   Wallet  |                                    | Credential Issuer |
+--------------+   +-----------+                                    +-------------------+
        |                |                                                    |
        |                |  (1) End-User provides  information required       |
        |                |      for the issuance of a certain Credential      |
        |-------------------------------------------------------------------->|
        |                |                                                    |
        |                |  (2) Credential Offer (Pre-Authorized Code)        |
        |                |<---------------------------------------------------|
        |                |  (3) Obtains Issuer's Credential Issuer metadata   |
        |                |<-------------------------------------------------->|
        |   interacts    |                                                    |
        |--------------->|                                                    |
        |                |                                                    |
        |                |  (4) Token Request (Pre-Authorized Code, tx_code)  |
        |                |--------------------------------------------------->|
        |                |      Token Response (access_token)                 |
        |                |<---------------------------------------------------|
        |                |                                                    |
        |                |  (5) Credential Request (access_token, proof(s))   |
        |                |--------------------------------------------------->| 
        |                |      Credential Response                           |
        |                |      (Credential(s))                               |
        |                |<---------------------------------------------------|             
~~~
!---
Figure: Issuance using Pre-Authorized Code Flow 

(1) The Credential Issuer successfully obtains consent and End-User data required for the issuance of a requested Credential from the End-User using Issuer-specific business process.

(2) The flow defined in this specification begins as the Credential Issuer generates a Credential Offer for certain Credential(s) and communicates it to the Wallet, for example, as a QR code or as a URI. The Credential Offer contains the Credential Issuer's URL, the information about the Credential(s) being offered and the Pre-Authorized Code. This step is defined in (#credential_offer).

(3) The Wallet uses the Credential Issuer's URL to fetch its metadata as described in (#credential_issuer_metadata). The Wallet needs the metadata to learn the Credential types and formats that the Credential Issuer supports, and to determine the Authorization Endpoint (OAuth 2.0 Authorization Server) as well as Credential Endpoint required to start the request.

(4) The Wallet sends the Pre-Authorized Code obtained in step (2) in the Token Request to the Token Endpoint. The Wallet will additionally send a Transaction Code provided by the User, if it was required by the Credential Issuer. This step is defined in (#token_endpoint).

(5) This step is the same as Step 5 in the Authorization Code Flow.

It is important to note that anyone who possesses a valid Pre-Authorized Code, without further security measures, would be able to receive a VC from the Credential Issuer. Implementers MUST implement mitigations most suitable to the use case.

One such mechanism defined in this specification is the usage of Transaction Codes. The Credential Issuer indicates the usage of Transaction Codes in the Credential Offer and sends the Transaction Code to the End-User via a second channel different than the issuance flow. After the End-User provides the Transaction Code, the Wallet sends the Transaction Code within the Token Request, and the Authorization Server verifies the Transaction Code.

For more details and concrete mitigations, see (#security_considerations_pre-authz-code).

# Credential Offer Endpoint {#credential_offer_endpoint}

This endpoint is used by a Credential Issuer in case it is already in an interaction with the End-User that wishes to initiate a Credential issuance. It is used to pass available information relevant for the Credential issuance to ensure a convenient and secure process.

## Credential Offer {#credential_offer}

The Credential Issuer sends Credential Offer using an HTTP GET request or an HTTP redirect to the Wallet's Credential Offer Endpoint defined in (#client_metadata).

The Credential Offer object, which is a JSON-encoded object with the Credential Offer parameters, can be sent by value or by reference.

The Credential Offer contains a single URI query parameter `credential_offer` or `credential_offer_uri`:

* `credential_offer`: Object with the Credential Offer parameters. This MUST NOT be present when `credential_offer_uri` parameter is present.
* `credential_offer_uri`: String that is a URL using the `https` scheme referencing a resource containing a JSON-encoded object with the Credential Offer parameters. This MUST NOT be present when `credential_offer` parameter is present.

The Credential Issuer MAY render a QR code containing the Credential Offer that can be scanned by the End-User using a Wallet, or a link that the End-User can click.

For security considerations, see (#credential-offer-security).

### Credential Offer Parameters {#credential_offer_parameters}

This specification defines the following parameters for the JSON-encoded Credential Offer object:

* `credential_issuer`: REQUIRED. The URL of the Credential Issuer, as defined in (#credential_issuer_identifier), from which the Wallet is requested to obtain one or more Credentials. The Wallet uses it to obtain the Credential Issuer's Metadata following the steps defined in (#credential_issuer_wellknown).
* `credential_configurations`: REQUIRED. Array of unique strings that each identify one of the keys in the name/value pairs stored in the `credential_configurations_supported` Credential Issuer metadata. The Wallet uses these string values to obtain the respective object that contains information about the Credential being offered as defined in (#credential-issuer-parameters). For example, these string values can be used to obtain `scope` value to be used in the Authorization Request.
* `grants`: OPTIONAL. Object indicating to the Wallet the Grant Types the Credential Issuer's AS is prepared to process for this Credential Offer. Every grant is represented by a name/value pair. The name is the Grant Type identifier; the value is an object that contains parameters either determining the way the Wallet MUST use the particular grant and/or parameters the Wallet MUST send with the respective request(s). If `grants` is not present or empty, the Wallet MUST determine the Grant Types the Credential Issuer's AS supports using the respective metadata. When multiple grants are present, it is at the Wallet's discretion which one to use.

The following values are defined by this specification: 

* Grant Type `authorization_code`:
  * `issuer_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the Wallet that is used to bind the subsequent Authorization Request with the Credential Issuer to a context set up during previous steps. If the Wallet decides to use the Authorization Code Flow and received a value for this parameter, it MUST include it in the subsequent Authorization Request to the Credential Issuer as the `issuer_state` parameter value.
  * `authorization_server`: OPTIONAL string that the Wallet can use to identify the Authorization Server to use with this grant type when `authorization_servers` parameter in the Credential Issuer metadata has multiple entries. It MUST NOT be used otherwise. The value of this parameter MUST match with one of the values in the `authorization_servers` array obtained from the Credential Issuer metadata.
* Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`:
  * `pre-authorized_code`: REQUIRED. The code representing the Credential Issuer's authorization for the Wallet to obtain Credentials of a certain type. This code MUST be short lived and single use. If the Wallet decides to use the Pre-Authorized Code Flow, this parameter value MUST be included in the subsequent Token Request with the Pre-Authorized Code Flow.
  * `tx_code`: OPTIONAL. Object specifying whether the AS expects presentation of a Transaction Code by the End-User along with the Token Request in a Pre-Authorized Code Flow. If the AS does not expect a Transaction Code, this object is absent, this is the default. The Transaction Code is intended to bind the Pre-Authorized Code to a certain transaction to prevent replay of this code by an attacker that, for example, scanned the QR code while standing behind the legitimate End-User. It is RECOMMENDED to send the Transaction Code via a separate channel. If the Wallet decides to use the Pre-Authorized Code Flow, the Transaction Code value MUST be sent in the `tx_code` parameter with the respective Token Request as defined in (#token_request). If no `length` or `description` is given, this object may be empty, indicating that a Transaction Code is required.
    * `input_mode` : OPTIONAL. String specifying the input characters set. Possible values are `numeric` (only digits) and `text` (any character). The default is `numeric`.
    * `length`: OPTIONAL. Integer specifying the length of the Transaction Code. This helps the Wallet to render the input screen and improve the user experience.
    * `description`: OPTIONAL. String containing guidance for the Holder of the Wallet on how to obtain the Transaction Code, e.g. describing over which communication channel it is delivered. The Wallet is RECOMMENDED to display this description next to the Transaction Code input screen to improve the user experience. The length of the string MUST NOT exceed 300 characters. The `description` does not support internationalization (i18n), however the Issuer MAY detect the Holder's language by previous communication or an HTTP Accept-Language header within an HTTP GET request for a Credential Offer URI.
  * `interval`: OPTIONAL. The minimum amount of time in seconds that the Wallet SHOULD wait between polling requests to the token endpoint (in case the Authorization Server responds with error code `authorization_pending` - see (#token_error_response)). If no value is provided, Wallets MUST use `5` as the default.
  * `authorization_server`: OPTIONAL string that the Wallet can use to identify the Authorization Server to use with this grant type when `authorization_servers` parameter in the Credential Issuer metadata has multiple entries. It MUST NOT be used otherwise. The value of this parameter MUST match with one of the values in the `authorization_servers` array obtained from the Credential Issuer metadata.
  
The following non-normative example shows a Credential Offer object where the Credential Issuer can offer the issuance of two different Credentials (which may be even of different formats):

<{{examples/credential_offer_multiple_credentials.json}}

### Sending Credential Offer by Value Using `credential_offer` Parameter

Below is a non-normative example of a Credential Offer passed by value:

```
  GET /credential_offer?credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.example.com%22,%22credentials%22:%5B%22UniversityDegree_JWT%22,%22org.iso.18013.5.1.mDL%22%5D,%22grants%22:%7B%22urn:ietf:params:oauth:grant-type:pre-authorized_code%22:%7B%22pre-authorized_code%22:%22oaKazRN8I0IbtZ0C7JuMn5%22,%22tx_code%22:%7B%7D%7D%7D%7D
```

The following is a non-normative example of a Credential Offer that can be included in a QR code or a link used to invoke a Wallet deployed as a native app:

```
openid-credential-offer://?credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.example.com%22,%22credentials%22:%5B%22org.iso.18013.5.1.mDL%22%5D,%22grants%22:%7B%22urn:ietf:params:oauth:grant-type:pre-authorized_code%22:%7B%22pre-authorized_code%22:%22oaKazRN8I0IbtZ0C7JuMn5%22,%22tx_code%22:%7B%22input_mode%22:%22text%22,%22description%22:%22Please%20enter%20the%20serial%20number%20of%20your%20physical%20drivers%20license%22%7D%7D%7D%7D
```

### Sending Credential Offer by Reference Using `credential_offer_uri` Parameter

Upon receipt of the `credential_offer_uri`, the Wallet MUST send an HTTP GET request to URI to retrieve the referenced Credential Offer Object, unless it is already cached, and parse it to recreate the Credential Offer parameters.

Note: The Credential Issuer SHOULD use a unique URI for each Credential Offer utilizing distinct parameters, or otherwise prevent the Credential Issuer from caching the `credential_offer_uri`.

Below is a non-normative example of this fetch process:

```
GET /credential_offer HTTP/1.1
Host: server.example.com
```

Response from the Credential Issuer that contains a Credential Offer Object MUST have the media type "application/json".

This ability to pass Credential Offer by reference is particularly useful for large requests.

Below is a non-normative example of the Credential Offer displayed by the Credential Issuer as a QR code when the Credential Offer is passed by reference:

```
openid-credential-offer://?
  credential_offer_uri=https%3A%2F%2Fserver%2Eexample%2Ecom%2Fcredential-offer.json
```

Below is a non-normative example of a response from the Credential Issuer that contains a Credential Offer Object used to encourage the Wallet to start an Authorization Code Flow:

<{{examples/credential_offer_authz_code.txt}}

Below is a non-normative example of a Credential Offer Object for a Pre-Authorized Code Flow (with a Credential type reference):

<{{examples/credential_offer_by_reference.json}}

When retrieving Credential Offer from the Credential Offer URL, `application/json` media type MUST be used. The Credential Offer cannot be signed and MUST NOT use `application/jwt` with `"alg": "none"`.

## Credential Offer Response

The Wallet is not supposed to create a response. UX control stays with the Wallet after completion of the process. 

# Authorization Endpoint {#authorization_endpoint}

The Authorization Endpoint is used in the same manner as defined in [@!RFC6749], taking into account the recommendations given in [@!I-D.ietf-oauth-security-topics].

When the grant type `authorization_code` is used, it is RECOMMENDED to use PKCE [@!RFC7636] and Pushed Authorization Requests [@RFC9126]. PKCE prevents authorization code interception attacks. Pushed Authorization Requests ensure the integrity and authenticity of the authorization request.

## Authorization Request {#credential-authz-request}

An Authorization Request is an OAuth 2.0 Authorization Request as defined in Section 4.1.1 of [@!RFC6749], which requests that access be granted to the Credential Endpoint, as defined in (#credential-endpoint).

There are two possible ways to request issuance of a specific Credential type in an Authorization Request. One way is to use the `authorization_details` request parameter, as defined in [@!RFC9396], with one or more authorization details objects of type `openid_credential`, per (#authorization-details). The other is through the use of scopes as defined in (#credential-request-using-type-specific-scope).

### Request Issuance of a Certain Credential Type using `authorization_details` Parameter {#authorization-details}

The request parameter `authorization_details` defined in Section 2 of [@!RFC9396] MUST be used to convey the details about the Credentials the Wallet wants to obtain. This specification introduces a new authorization details type `openid_credential` and defines the following parameters to be used with this authorization details type:

* `type`: REQUIRED. String that determines the authorization details type. MUST be set to `openid_credential` for the purpose of this specification.
* `credential_configuration_id`: REQUIRED. String specifying a unique identifier of the Credential being described in the `credential_configurations_supported` map in the Credential Issuer Metadata as defined in (#credential-issuer-parameters). The referenced object in the `credential_configurations_supported` map conveys the details, such as the format, for issuance of the requested Credential. This specification defines Credential Format specific Issuer Metadata in (#format_profiles).

The following is a non-normative example of an `authorization_details` object:

<{{examples/authorization_details.json}}

If the Credential Issuer metadata contains an `authorization_servers` parameter, the authorization detail's `locations` common data field MUST be set to the Credential Issuer Identifier value. A non-normative example for a deployment where an AS protects multiple Credential Issuers would look like this:

<{{examples/authorization_details_with_as.json}}

Below is a non-normative example of an Authorization Request using the `authorization_details` parameter that would be sent by the User Agent to the Authorization Server in response to an HTTP 302 redirect response by the Wallet (with line wraps within values for display purposes only):

```
GET /authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &authorization_details=%5B%7B%22type%22%3A%20%22openid_credential%22%2C%20%22
    credential_configuration_id%22%3A%20%22UniversityDegreeCredential%22%7D%5D
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
  
Host: https://server.example.com
```

This non-normative example requests authorization to issue two different Credentials:

<{{examples/authorization_details_multiple_credentials.json}}

Note: Applications MAY combine authorization details of type `openid_credential` with any other authorization details types in an Authorization Request.

### Using `scope` Parameter to Request Issuance of a Credential {#credential-request-using-type-specific-scope}

In addition to a mechanism defined in (#credential-authz-request), Credential Issuers MAY support requesting authorization to issue a Credential using the OAuth 2.0 `scope` parameter.

When the Wallet does not know which scope value to use to request issuance of a certain Credential, it can discover it using the `scope` Credential Issuer metadata parameter defined in (#credential-issuer-parameters). When the flow starts with a Credential Offer, the Wallet can use the `credential_configurations` parameter values to identify object(s) in the `credential_configurations_supported` map in the Credential Issuer metadata parameter and use the `scope` parameter value from that object.

The Wallet can discover the scope values using other options such as normative text in a profile of this specification that defines scope values along with a description of their semantics.

The concrete `scope` values are out of scope of this specification.

The Wallet MAY combine scopes discovered from the Credential Issuer metadata with the scopes discovered from the Authorization Server metadata.

It is RECOMMENDED to use collision-resistant scope values.

Credential Issuers MUST interpret each scope value as a request to access the Credential Endpoint as defined in (#credential-endpoint) for the issuance of a Credential type identified by that scope value. Multiple scope values MAY be present in a single request whereby each 
occurrence MUST be interpreted individually.

Credential Issuers MUST ignore unknown scope values in a request.

If the Credential Issuer metadata contains an `authorization_servers` property, it is RECOMMENDED to use a `resource` parameter [@!RFC8707] whose value is the Credential Issuer's identifier value to allow the AS to differentiate Credential Issuers.  

Below is a non-normative example of an Authorization Request provided by the Wallet to the Authorization Server using the scope `UniversityDegree_JWT` and in response to an HTTP 302 redirect. The line wraps within the values are for display purposes only:

```
GET /authorize?
  response_type=code
  &scope=UniversityDegreeCredential
  &resource=https://credential-issuer.example.com
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
Host: https://server.example.com
```

If a scope value related to Credential issuance and the `authorization_details` request parameter containing objects of type `openid_credential` are both present in a single request, the Credential Issuer MUST interpret these individually. However, if both request the same Credential type, then the Credential Issuer MUST follow the request as given by the authorization details object.

### Additional Request Parameters {#additional_request_parameters}

This specification defines the following request parameters that can be supplied in an Authorization Request:

* `wallet_issuer`: OPTIONAL. String containing the Wallet's identifier. The Credential Issuer can use the discovery process defined in [@!SIOPv2] to determine the Wallet's capabilities and endpoints, using the `wallet_issuer` value as the Issuer Identifier referred to in [@!SIOPv2]. This is RECOMMENDED in Dynamic Credential Requests.
* `user_hint`: OPTIONAL. String containing an opaque End-User hint that the Wallet MAY use in subsequent callbacks to optimize the End-User's experience. This is RECOMMENDED in Dynamic Credential Requests.
* `issuer_state`: OPTIONAL. String value identifying a certain processing context at the Credential Issuer. A value for this parameter is typically passed in a Credential Offer from the Credential Issuer to the Wallet (see (#credential_offer)). This request parameter is used to pass the `issuer_state` value back to the Credential Issuer.

Note: When processing the Authorization Request, the Credential Issuer MUST take into account that the `issuer_state` is not guaranteed to originate from this Credential Issuer in all circumstances. It could have been injected by an attacker.

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

To enable dynamic callbacks of the Credential Issuer to the End-User's Wallet, the Wallet MAY provide the additional parameters `wallet_issuer` and `user_hint` defined in (#additional_request_parameters).

For non-normative examples of the request and response, see Section TBD in [@OpenID4VP].

Note to the editors: We need to sort out the Credential Issuer's `client_id` with the Wallet and potentially add an example with `wallet_issuer` and `user_hint`.

## Successful Authorization Response {#authorization_response}

Authorization Responses MUST be made as defined in [@!RFC6749].

Below is a non-normative example of a successful Authorization Response:

```
HTTP/1.1 302 Found
Location: https://Wallet.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
```

## Authorization Error Response

Authorization Error Responses MUST be made as defined in [@!RFC6749].

When the requested scope value is invalid, unknown, or malformed, the AS should respond with the error code `invalid_scope` defined in Section 4.1.2.1 of [@!RFC6749].

Below is a non-normative example of an unsuccessful Authorization Response.

```json=
HTTP/1.1 302 Found
Location: https://client.example.net/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value
```

# Token Endpoint {#token_endpoint}

The Token Endpoint issues an Access Token and, optionally, a Refresh Token in exchange for the Authorization Code that Client obtained in a successful Authorization Response. It is used in the same manner as defined in [@!RFC6749] and follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

## Token Request {#token_request}

Upon receiving a successful Authorization Response, a Token Request is made as defined in Section 4.1.3 of [@!RFC6749].

The following are the extension parameters to the Token Request used in the Pre-Authorized Code Flow defined in (#pre-authz-code-flow):

* `pre-authorized_code`: The code representing the authorization to obtain Credentials of a certain type. This parameter MUST be present if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.
* `tx_code`: OPTIONAL. String value containing a Transaction Code. This value MUST be present if a `tx_code` object was present in the Credential Offer (including if the object was empty). This parameter MUST only be used if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.

Requirements around how the Wallet identifies and, if applicable, authenticates itself with the Authorization Server in the Token Request depends on the Client type defined in Section 2.1 of [@!RFC6749] and the Client authentication method indicated in the `token_endpoint_auth_method` Client metadata. The requirements specified in Sections 4.1.3 and 3.2.1 of [@!RFC6749] MUST be followed.

For the Pre-Authorized Code Grant Type, authentication of the Client is OPTIONAL, as described in Section 3.2.1 of OAuth 2.0 [@!RFC6749], and, consequently, the `client_id` parameter is only needed when a form of Client Authentication that relies on this parameter is used.

If the Token Request contains an `authorization_details` parameter (as defined by [RFC@!9396]) of type `openid_credential` and the Credential Issuer's metadata contains an `authorization_servers` parameter, the `authorization_details` object MUST contain the Credential Issuer's identifier in the `locations` element. 

If the Token Request contains a scope value related to Credential issuance and the Credential Issuer's metadata contains an `authorization_servers` parameter, it is RECOMMENDED to use a `resource` parameter [@!RFC8707] whose value is the Credential Issuer's identifier value to allow the AS to differentiate Credential Issuers.

When the Pre-Authorized Grant Type is used, it is RECOMMENDED that the Credential Issuer issues an Access Token valid only for the Credentials indicated in the Credential Offer (see (#credential_offer)). The Wallet SHOULD obtain a separate Access Token if it wants to request issuance of any the Credentials that were not included in the Credential Offer, but were discoverable from the Credential Issuer's `credential_configurations_supported` metadata parameter.

Below is a non-normative example of a Token Request in an Authorization Code Flow:

```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

grant_type=authorization_code
&code=SplxlOBeZQQYbYS6WxSbIA
&code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
&redirect_uri=https%3A%2F%2FWallet.example.org%2Fcb  
```

Below is a non-normative example of a Token Request in a Pre-Authorized Code Flow (without Client Authentication):

```
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
&pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
&tx_code=493536
```

## Successful Token Response {#token-response}

Token Responses are made as defined in [@!RFC6749].

The AS might decide to authorize issuance of multiple instances for each Credential requested in the Authorization Request. Each Credential instance is described using the same entry in the `credential_configurations_supported` Credential Issuer metadata, but contains different claim values or a different subset of claims within the claims set identified by the Credential description.

In addition to the response parameters defined in [@!RFC6749], the AS MAY return the following parameters:

* `c_nonce`: OPTIONAL. String containing a nonce to be used when creating a proof of possession of the key proof (see (#credential_request)). When received, the Wallet MUST use this nonce value for its subsequent requests until the Credential Issuer provides a fresh nonce.
* `c_nonce_expires_in`: OPTIONAL. Number denoting the lifetime in seconds of the `c_nonce`.
* `authorization_details`: REQUIRED when `authorization_details` parameter is used to request issuance of a certain Credential type as defined in (#authorization-details). It MUST NOT be used otherwise. It is an array of objects, as defined in Section 7 of [@!RFC9396]. In addition to the parameters defined in (#authorization-details), this specification defines the following parameter to be used with the authorization details type `openid_credential` in the Token Response:
  * `credential_identifiers`: OPTIONAL. Array of strings, each uniquely identifying a Credential that can be issued using the Access Token returned in this response. Each of these Credentials corresponds to the same entry in the `credential_configurations_supported` Credential Issuer metadata but can contain different claim values or a different subset of claims within the claimset identified by that Credential type. This parameter can be used to simplify the Credential Request, as defined in (#credential_request), where the `credential_identifier` parameter replaces the `format` parameter and any other Credential format-specific parameters in the Credential Request. When received, the Wallet MUST use these values together with an Access Token in subsequent Credential Requests.

Note: Credential Instance identifier(s) cannot be used when the `scope` parameter is used in the Authorization Request to request issuance of a Credential.

Below is a non-normative example of a Token Response when the `authorization_details` parameter was used to request issuance of a certain Credential type:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

  {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "token_type": "bearer",
    "expires_in": 86400,
    "c_nonce": "tZignsnFbp",
    "c_nonce_expires_in": 86400,
    "authorization_details": [
      {
        "type": "openid_credential",
        "credential_configuration_id": "UniversityDegreeCredential",
        "credential_identifiers": [ "CivilEngineeringDegree-2023", "ElectricalEngineeringDegree-2023" ]
      }
    ]
  }
```

## Token Error Response {#token_error_response}

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

This specification also uses the error codes `authorization_pending` and `slow_down` defined in [@!RFC8628] for the Pre-Authorized Code grant type:

 `authorization_pending`:

- This error code is used if the Authorization Server is waiting for an End-User interaction or a downstream process to complete. The Wallet SHOULD repeat the access token request to the token endpoint (a process known as polling). Before each new request, the Wallet MUST wait at least the number of seconds specified by the `interval` claim of the Credential Offer (see (#credential_offer_parameters)) or the authorization response (see (#authorization_response)), or 5 seconds if none was provided, and respect any increase in the polling interval required by the "slow_down" error.

`slow_down`:

- A variant of `authorization_pending` error code, the authorization request is still pending and polling should continue, but the interval MUST be increased by 5 seconds for this and all subsequent requests.

# Credential Endpoint {#credential-endpoint}

The Credential Endpoint issues a Credential as approved by the End-User upon presentation of a valid Access Token representing this approval. Support for this endpoint is REQUIRED.

Communication with the Credential Endpoint MUST utilize TLS. 

The Client can request issuance of a Credential of a certain type multiple times, e.g., to associate the Credential with different public keys/Decentralized Identifiers (DIDs) or to refresh a certain Credential.

If the Access Token is valid for requesting issuance of multiple Credentials, it is at the Client's discretion to decide the order in which to request issuance of multiple Credentials requested in the Authorization Request.

## Binding the Issued Credential to the Identifier of the End-User Possessing that Credential {#credential-binding}

Issued Credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the Credential. Cryptographic binding allows the Verifier to verify during the presentation of a Credential that the End-User presenting a Credential is the same End-User to whom that Credential was issued. For non-cryptographic type of binding and Credentials issued without any binding, see Implementation Considerations in (#claim-based-binding) and (#no-binding).

Note: Claims in the Credential are usually about the End-User who possesses it, but can be about another entity.

For cryptographic binding, the Client has the following options to provide cryptographic binding material for a requested Credential as defined in (#credential_request):

1. Provide proof of control alongside key material (`proof` that includes `sub_jwk` or `did`)
1. Provide only proof of control without the key material (`proof` that does not include `sub_jwk` or `did`)

## Credential Request {#credential_request}

A Client makes a Credential Request to the Credential Endpoint by sending the following parameters in the entity-body of an HTTP POST request using the `application/json` media type.

* `format`: REQUIRED when the `credential_identifier` was not returned from the Token Response. MUST NOT be used otherwise. String that determines the format of the Credential to be issued, which may determine the type and any other information related to the Credential to be issued. Credential Format Profiles consisting of the Credential format specific set of parameters are defined in (#format_profiles). When this parameter is used, `credential_identifier` parameter MUST NOT be present.
* `proof`: OPTIONAL. Object containing the proof of possession of the cryptographic key material the issued Credential would be bound to.  The `proof` object is REQUIRED if the `proof_types` parameter is non-empty and present in the `credential_configurations_supported` map of the Issuer metadata for the requested Credential. The `proof` object MUST contain a following claim:
    * `proof_type`: REQUIRED. String denoting the key proof type. The value of this claim determines other claims in the key proof object and its respective processing rules. Key proof types defined in this specification can be found in (#proof_types).
* `credential_identifier`: REQUIRED when `credential_identifier` was returned from the Token Response. MUST NOT be used otherwise. String that identifies a Credential that is being requested to be issued. When this parameter is used, the `format` parameter and any other Credential format specific set of parameters such as those defined in (#format_profiles) MUST NOT be present.
* `credential_response_encryption`: OPTIONAL. Object containing information for encrypting the Credential Response. If this request element is not present, the corresponding credential response returned is not encrypted.
    * `jwk`: REQUIRED. Object containing a single public key as a JWK used for encrypting the Credential Response.
    * `alg`: REQUIRED. JWE [@!RFC7516] `alg` algorithm [@!RFC7518] for encrypting Credential Responses.
    * `enc`: REQUIRED. JWE [@!RFC7516] `enc` algorithm [@!RFC7518] for encrypting Credential Responses.

The `proof_type` claim is an extension point that enables the use of different types of proofs for different cryptographic schemes.

The `proof` element MUST incorporate the Credential Issuer Identifier (audience), and a `c_nonce` value generated by the Authorization Server or the Credential Issuer to allow the Credential Issuer to detect replay. The way that data is incorporated depends on the key proof type. In a JWT, for example, the `c_nonce` is conveyed in the `nonce` claims whereas the audience is conveyed in the `aud` claim. In a Linked Data proof, for example, the `c_nonce` is included as the `challenge` element in the key proof object and the Credential Issuer (the intended audience) is included as the `domain` element.

Initial `c_nonce` value can be returned in a successful Token Response as defined in (#token-response), in a Credential Error Response as defined in (#issuer-provided-nonce), or in a Batch Credential Error Response as defined in (#batch-credential_error_response).

Below is a non-normative example of a Credential Request for a Credential in [@ISO.18013-5] format using Credential format specific set of parameters and a key proof type `cwt`:

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
   "format":"mso_mdoc",
   "doctype":"org.iso.18013.5.1.mDL",
   "proof": {
      "proof_type": "cwt",
      "cwt": "..."
   }
}
```

Below is a non-normative example of a Credential Request for a Credential in an IETF SD-JWT VC format using Credential instance identifier and a key proof type `jwt`:

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
   "credential_identifier": "CivilEngineeringDegree-2023",
   "proof": {
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
}
```

The Client MAY request encrypted responses by providing its encryption parameters in the Credential Request.

The Credential Issuer MAY require encrypted responses by including `require_credential_response_encryption` in the Credntial Issuer Metadata.

### Key Proof Types {#proof_types}

This specification defines the following values for the `proof_type` property:

* `jwt`: A JWT [@!RFC7519] is used as proof of possession. When `proof_type` is `jwt`, a `proof` object MUST include a `jwt` claim containing a JWT defined in (#jwt-proof-type).
* `cwt`: A CWT [@!RFC8392] is used as proof of possession. When `proof_type` is `cwt`, a `proof` object MUST include a `cwt` claim containing a CWT defined in (#cwt-proof-type).
* `ldp_vp`: A W3C Verifiable Presentation object signed using the Data Integrity Proof as defined in [@VC_DATA_2.0] or [@VC_DATA], and where the proof of possession MUST be done in accordance with [@Data_Integrity]. When `proof_type` is set to `ldp_vp`, the `proof` object MUST include a `ldp_vp` claim containing a [W3C Verifiable Presentation](https://www.w3.org/TR/vc-data-model-2.0/#presentations-0) defined in (#ldp_vp-proof-type).

#### `jwt` Key Proof Type {#jwt-proof-type}

The JWT MUST contain the following elements:

* in the JOSE header,
  * `alg`: REQUIRED. A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry [@IANA.JOSE.ALGS]. MUST NOT be `none` or an identifier for a symmetric algorithm (MAC).
  * `typ`: REQUIRED. MUST be `openid4vci-proof+jwt`, which explicitly types the key proof JWT as recommended in Section 3.11 of [@!RFC8725].
  * `kid`: OPTIONAL. JOSE Header containing the key ID. If the Credential shall be bound to a DID, the `kid` refers to a DID URL which identifies a particular key in the DID Document that the Credential shall be bound to. It MUST NOT be present if `jwk` is present.
  * `jwk`: OPTIONAL. JOSE Header containing the key material the new Credential shall be bound to. It MUST NOT be present if `kid` is present.
  * `x5c`: OPTIONAL. JOSE Header containing a certificate or certificate chain corresponding to the key used to sign the JWT. This element MAY be used to convey a key attestation. In such a case, the actual key certificate will contain attributes related to the key properties.
  * `trust_chain`: OPTIONAL. JOSE Header containing an [@!OpenID.Federation] Trust Chain. This element MAY be used to convey key attestation, metadata, metadata policies, federation Trust Marks and any other information related to a specific federation, if available in the chain. When used for signature verification, the header parameter `kid` MUST be present.

* in the JWT body,
  * `iss`: OPTIONAL (string). The value of this claim MUST be the `client_id` of the Client making the Credential request. This claim MUST be omitted if the access token authorizing the issuance call was obtained from a Pre-Authorized Code Flow through anonymous access to the token endpoint.
  * `aud`: REQUIRED (string). The value of this claim MUST be the Credential Issuer Identifier.
  * `iat`: REQUIRED (number). The value of this claim MUST be the time at which the key proof was issued using the syntax defined in [@!RFC7519].
  * `nonce`: OPTIONAL (string). The value type of this claim MUST be a string, where the value is a server-provided `c_nonce`. MUST be present when the Wallet received server-provided `c_nonce`.

The Credential Issuer MUST validate that the `proof` is actually signed by a key identified in the JOSE Header.

Below is a non-normative example of a `proof` parameter (line breaks for display purposes only):

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

where the JWT looks like this:

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

#### `ldp_vp` Key Proof Type {#ldp_vp-proof-type}

When a W3C Verifiable Presentation as defined by [@VC_DATA_2.0] or [@VC_DATA] signed using Data Integrity is used as Key Proof, it MUST contain the following elements:

* `holder`: OPTIONAL. MUST be equivalent to the controller identifier (e.g. DID) for the `verificationMethod` value identified by the `proof.verificationMethod` property.
* `proof`: REQUIRED. The proof body of a W3C Verifiable Presentation.
  * `domain`: REQUIRED (string). The value of this claim MUST be the Credential Issuer Identifier.
  * `challenge`: REQUIRED when the Credential Issuer has provided a `c_nonce`. MUST NOT be used otherwise. String, where the value is a server-provided `c_nonce`. It MUST be present when the Wallet received server-provided `c_nonce`.

The Credential Issuer MUST validate that the `proof` is actually signed with a key in possession of the Holder.

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

#### `cwt` Key Proof Type {#cwt-proof-type}

The CWT MUST contain the following elements:

* in the COSE protected header (see [@!RFC8152], Section 3.1.),
  * Label 1 (`alg`): REQUIRED. A digital signature algorithm identifier such as per IANA "COSE Algorithms" registry [@IANA.COSE.ALGS]. MUST NOT be an identifier for a symmetric algorithm (MAC).
  * Label 3 (`content type`): REQUIRED. MUST be `openid4vci-proof+cwt`, which explicitly types the key proof CWT.
  * (string-valued) Label `COSE_Key`: OPTIONAL (byte string). COSE key material the new Credential shall be bound to. It MUST NOT be present if `x5chain` is present.
  * Label 33 (`x5chain`): OPTIONAL (byte string). As defined in [@!RFC9360], contains an ordered array of X.509 certificates corresponding to the key used to sign the CWT. It MUST NOT be present if `COSE_Key` is present.
* in the content of the message (see [@!RFC8392], Section 4),
  * Claim Key 1 (`iss`): OPTIONAL (text string). The value of this claim MUST be the `client_id` of the Client making the Credential request. This claim MUST be omitted if the access token authorizing the issuance call was obtained from a Pre-Authorized Code Flow through anonymous access to the token endpoint.
  * Claim Key 3 (`aud`): REQUIRED (text string). The value of this claim MUST be the Credential Issuer Identifier.
  * Claim Key 6 (`iat`): REQUIRED (integer or floating-point number). The value of this claim MUST be the time at which the key proof was issued.
  * Claim Key 10 (`Nonce`): OPTIONAL (byte string). The value of this claim MUST be a server-provided `c_nonce` converted from string to bytes. MUST be present when the Wallet received server-provided `c_nonce`.

### Verifying Key Proof {#verifying-key-proof}

To validate a Key Proof, the Credential Issuer MUST ensure that:

- all required claims for that proof type are contained as defined in (#proof_types),
- the Key Proof is explicitly typed using header parameters as defined for that proof type,
- the header parameter indicates a registered asymmetric digital signature algorithm, `alg` parameter value is not `none`, is supported by the application, and is acceptable per local policy,
- the signature on the Key Proof verifies with the public key contained in the header parameter,
- the header parameter does not contain a private key,
- the `nonce` claim (or Claim Key 10) matches the server-provided `c_nonce` value, if the server had previously provided a `c_nonce`,
- the creation time of the JWT, as determined by either the issuance time, or a server managed timestamp via the nonce claim, is within an acceptable window (see (#key-proof-replay)).

These checks may be performed in any order.

## Credential Response {#credential-response}

Credential Response can be immediate or deferred. The Credential Issuer MAY be able to immediately issue a requested Credential and send it to the Client.

In other cases, the Credential Issuer MAY NOT be able to immediately issue a requested Credential and would want to send a `transaction_id` parameter to the Client to be used later to receive a Credential when it is ready. The HTTP status code MUST be 202 (section 10.2.3 of [@!RFC2616]).

If the Client requested an encrypted response by including the `credential_response_encryption` object in the request, the Credential Issuer MUST encode the information in the Credential Response as a JWT using the  parameters from the `credential_response_encryption` object. If the Credential Response is encrypted, the media type of the response MUST bet set to `application/jwt`. If encryption was requested in the Credential Request and the Credential Response is not encrypted, the Client SHOULD reject the Credential Response.

If the Credential Response is not encrypted, the media type of the response MUST be set to `application/json`.

The following claims are used in the JSON-encoded Credential Response body:

* `format`: REQUIRED. String denoting the format of the issued Credential.
* `credential`: OPTIONAL. Contains issued Credential. MUST be present when `transaction_id` is not returned. MAY be a string or an object, depending on the Credential format. See (#format_profiles) for the Credential format specific encoding requirements.
* `transaction_id`: OPTIONAL. String identifying a Deferred Issuance transaction. This claim is contained in the response if the Credential Issuer was unable to immediately issue the Credential. The value is subsequently used to obtain the respective Credential with the Deferred Credential Endpoint (see (#deferred-credential-issuance)). It MUST be present when the `credential` parameter is not returned. It MUST be invalidated after the Credential for which it was meant has been obtained by the Wallet.
* `c_nonce`: OPTIONAL. String containing a nonce to be used to create a proof of possession of key material when requesting a Credential (see (#credential_request)). When received, the Wallet MUST use this nonce value for its subsequent Credential Requests until the Credential Issuer provides a fresh nonce.
* `c_nonce_expires_in`: OPTIONAL. Number denoting the lifetime in seconds of the `c_nonce`.
* `notification_id`: OPTIONAL. String identifying an issued Credential that the Wallet includes in the Notification Request as defined in (#notification ). This parameter MUST NOT be present if `credential` parameter is not present.

The `format` key determines the Credential format and encoding of the credential in the Credential Response. Details are defined in the Credential Format Profiles in (#format_profiles). 

Credential formats expressed as binary data MUST be base64url-encoded and returned as a string.

Below is a non-normative example of a Credential Response in an immediate issuance flow for a Credential in JWT VC format (JSON encoded):

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "format": "jwt_vc_json",
  "credential": "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L",
  "c_nonce": "fGFF7UkhLa",
  "c_nonce_expires_in": 86400  
}
```

Below is a non-normative example of a Credential Response in a deferred flow:

```
HTTP/1.1 202 Accepted
Content-Type: application/json
Cache-Control: no-store

{
  "transaction_id": "8xLOxBtZp8",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400  
}
```

### Credential Error Response {#credential-error-response}

When the Credential Request is invalid or unauthorized, the Credential Issuer constructs the error response as defined in this section.

#### Authorization Errors {#authorization-errors}

If the Credential Request does not contain an Access Token that enables issuance of a requested Credential, the Credential Endpoint returns an authorization error response such as defined in section 3 of [@!RFC6750].

#### Credential Request Errors {#credential-request-errors}

For the errors specific to the payload of the Credential Request such as those caused by `type`, `format`, `proof`, or encryption parameters in the request, the error codes values defined in this section MUST be used instead of a generic `invalid_request` parameter defined in section 3.1 of [@!RFC6750].

If the Wallet is requesting the issuance of a Credential that is not supported by the Credential Endpoint, the HTTP response MUST use the HTTP status code 400 (Bad Request) and set the content type to `application/json` with the following parameters in the JSON-encoded response body:

* `error`: REQUIRED. A key at the top level of the object, the value of which SHOULD be a single ASCII [@!USASCII] error code from the following:
  * `invalid_credential_request`: The Credential Request is missing a required parameter, includes an unsupported parameter or parameter value, repeats the same parameter, or is otherwise malformed.
  * `unsupported_credential_type`: Requested Credential type is not supported.
  * `unsupported_credential_format`: Requested Credential format is not supported.
  * `invalid_proof`: The `proof` in the Credential Request is invalid. The `proof` field is not present or the provided key proof is invalid or not bound to a nonce provided by the Credential Issuer.
  * `invalid_encryption_parameters`: This error occurs when the encryption parameters in the Credential Request are either invalid or missing. In the latter case, it indicates that the Credential Issuer requires the Credential Response to be sent encrypted, but the Credential Request does not contain the necessary encryption parameters.
* `error_description`: OPTIONAL. A key at the top level of the object. The value MUST be a human-readable ASCII [@!USASCII] text, providing any additional information used to assist the Client implementers in understanding the occurred error. The values for the `error_description` parameter MUST NOT include characters outside the set `%x20-21 / %x23-5B / %x5D-7E`.

The usage of these parameters takes precedence over the `invalid_request` parameter defined in (#authorization-errors), since they provide more details about the errors.

The following is a non-normative example of a Credential Error Response where an unsupported Credential format was requested:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
   "error": "unsupported_credential_format"
}
```

### Credential Issuer Provided Nonce {#issuer-provided-nonce}

The Credential Issuer that requires the Client to send a key proof of possession of the key material for the Credential to be bound to (`proof`) MAY receive a Credential Request without such a key proof or with an invalid key proof. In such a case, the Credential Issuer MUST provide the Client with a `c_nonce` defined in (#credential-response) in a Credential Error Response using `invalid_proof` error code defined in (#credential-error-response). 

The `c_nonce` value MUST be incorporated in the respective parameter in the `proof` object.

Below is a non-normative example of a Credential Response when the Credential Issuer is requesting a Wallet to provide in a subsequent Credential Request a key proof that is bound to a `c_nonce`:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "invalid_proof"
  "error_description":
       "Credential Issuer requires key proof to be bound to a Credential Issuer provided nonce.",
  "c_nonce": "8YE9hCnyV2",
  "c_nonce_expires_in": 86400  
}
```

# Batch Credential Endpoint {#batch-credential-endpoint}

The Batch Credential Endpoint issues multiple Credentials in one Batch Credential Response as approved by the End-User upon presentation of a valid Access Token representing this approval. Support for this endpoint is OPTIONAL.

Communication with the Batch Credential Endpoint MUST utilize TLS. 

The Client can request issuance of multiple Credentials of certain types and formats in one Batch Credential Request. This includes Credentials of the same type and multiple formats, different types and one format, or both. 

## Batch Credential Request {#batch-credential_request}

The Batch Credential Endpoint allows a Client to send multiple Credential Request objects (see (#credential_request)) to request the issuance of multiple Credentials at once. A Batch Credential Request MUST be sent as a JSON-encoded object using the `application/json` media type.

The following keys are used in the Batch Credential Request:

* `credential_requests`: REQUIRED. Array that contains Credential Request objects as defined in (#credential_request).

Below is a non-normative example of a Batch Credential Request:

```
POST /batch_credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
   "credential_requests":[
      {
         "format":"jwt_vc_json",
         "credential_definition": {
           "type":[
             "VerifiableCredential",
             "UniversityDegreeCredential"
           ]
         },
         "proof":{
            "proof_type":"jwt",
            "jwt":"eyJ0eXAiOiJvcGVuaWQ0dmNpL...Lb9zioZoipdP-jvh1WlA"
         }
      },
      {
         "format":"mso_mdoc",
         "doctype":"org.iso.18013.5.1.mDL",
         "proof":{
            "proof_type":"jwt",
            "jwt":"eyJraWQiOiJkaWQ6ZXhhbXBsZ...KPxgihac0aW9EkL1nOzM"
         }
      }
   ]
}
```

## Batch Credential Response {#batch-credential_response}

A successful Batch Credential Response MUST contain all the requested Credentials. The Batch Credential Response MUST be sent as a JSON-encoded object using the `application/json` media type.

The following claims are used in the Batch Credential Response:

* `credential_responses`: REQUIRED. Array that contains Credential Response objects as defined in (#credential_request) and/or Deferred Credential Response objects as defined in (#deferred-credential_request). Every entry of the array corresponds to the Credential Request object at the same array index in the `credential_requests` parameter of the Batch Credential Request.
* `c_nonce`: OPTIONAL. The `c_nonce` as defined in (#credential-response). 
* `c_nonce_expires_in`: OPTIONAL. The `c_nonce_expires_in` as defined in (#credential-response).

Below is a non-normative example of a Batch Credential Response in an immediate issuance flow:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "credential_responses": [{
    "format": "jwt_vc_json",
    "credential": "eyJraWQiOiJkaWQ6ZXhhbXBsZTpl...C_aZKPxgihac0aW9EkL1nOzM"
  },
  {
    "format": "mso_mdoc",
    "credential": "YXNkZnNhZGZkamZqZGFza23....29tZTIzMjMyMzIzMjMy"
  }],
  "c_nonce": "fGFF7UkhLa",
  "c_nonce_expires_in": 86400
}
```

Below is a non-normative example of a Batch Credential Response that contains Credential Response and Deferred Credential Response objects:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
   "credential_responses":[
      {
         "transaction_id":"8xLOxBtZp8"
      },
      {
         "format":"jwt_vc_json",
         "credential":"YXNkZnNhZGZkamZqZGFza23....29tZTIzMjMyMzIzMjMy"
      }
   ],
   "c_nonce":"fGFF7UkhLa",
   "c_nonce_expires_in":86400
}
```

## Batch Credential Error Response {#batch-credential_error_response}

The Batch Credential Endpoint MUST respond with an HTTP 400 (Bad Request) status code in case of an error, unless specified otherwise.

Error codes extensions defined in (#credential-error-response) apply.

The Batch Credential Request MUST fail entirely if there is even one Credential failed to be issued. `transaction_id` MUST NOT be returned in this case.

When the Credential Issuer requires `proof` objects to be present in the Batch Credential Request, but does not receive them, it will return a Batch Credential Error Response with a `c_nonce` using `invalid_proof` error code as defined in (#issuer-provided-nonce).

# Deferred Credential Endpoint {#deferred-credential-issuance}

This endpoint is used to issue a Credential previously requested at the Credential Endpoint or Batch Credential Endpoint in cases where the Credential Issuer was not able to immediately issue this Credential. Support for this endpoint is OPTIONAL.

The Wallet MUST present to the Deferred Endpoint an Access Token valid for the issuance of the Credential previously requested at the Credential Endpoint or the Batch Credential Endpoint. 

Communication with the Deferred Credential Endpoint MUST utilize TLS. 

## Deferred Credential Request {#deferred-credential_request}

The Deferred Credential Request is an HTTP POST request. It MUST be sent using the `application/json` media type.

The following claims are used in the Batch Credential Response:

* `transaction_id`: REQUIRED. String identifying a Deferred Issuance transaction.

Credential Issuer MUST invalidate `transaction_id` after the Credential for which it was meant was obtained by the Wallet.

The following is a non-normative example of a Deferred Credential Request:

```
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
   "transaction_id":"8xLOxBtZp8"
}

```

## Deferred Credential Response {#deferred-credential_response}

The Deferred Credential Response uses the `format` and `credential` parameters as defined in (#credential-response).

Deferred Credential Response MUST be sent using the `application/json` media type.

## Deferred Credential Error Response {#deferred-credential_error_response}

When the Deferred Credential Request is invalid or the Credential is not available yet, the Credential Issuer constructs the error response as defined in (#credential-error-response).

The following additional error codes are specified in addition to those already defined in (#credential-request-errors):

* `issuance_pending` - The Credential issuance is still pending. The error response SHOULD also contain the `interval` member, determining the minimum amount of time in seconds that the Wallet needs to wait before providing a new request to the Deferred Credential Endpoint.  If `interval` member is missing or its value is not provided, the Wallet MUST use `5` as the default value.
* `invalid_transaction_id` - The Deferred Credential Request contains an invalid `transaction_id`. This error occurs when the `transaction_id` was not issued by the respective Credential Issuer or it was already used to obtain the Credential.

This is a non-normative example of a Deferred Credential Error Response:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
   "error": "invalid_transaction_id"
}
```

# Notification Endpoint {#notification_endpoint}

This endpoint is used by the Wallet to notify the Credential Issuer of certain events for issued Credentials. These events enable the Credential Issuer to take subsequent actions after issuance. The Credential Issuer needs to return one or more `notification_id` parameters in the Credential Response or the Batch Credential Response for the Wallet to be able to use this Endpoint. Support for this endpoint is OPTIONAL. The Issuer cannot assume that a notification will be sent for every issued credential since the use of this Endpoint is not mandatory for the Wallet.

The Wallet MUST present to the Notification Endpoint a valid Access Token issued at the Token Endpoint as defined in (#token_endpoint). 

Note: A Credential Issuer that requires a request to the Notification Endpoint MUST ensure the Access Token issued by the Authorization Server is valid at the Notification Endpoint.

The notification from the Wallet is idempotent. When the Credential Issuer receives multiple identical calls from the Wallet for the same `notification_id`, it returns success. Due to the network errors, there are no guarantees that a Credential Issuer receives a notification within a certain time period or at all.

Communication with the Notification Endpoint MUST utilize TLS.

## Notification Request {#notification}

The Wallet sends an HTTP POST request to the Notification Endpoint with the following parameters in the entity-body and using the `application/json` media type. If the Wallet supports the Notification Endpoint, the Wallet MAY send one or more Notification Requests per Credential issued.

* `notification_id`: REQUIRED. String received in Credential Response or Batch Credential Response.
* `event`: REQUIRED. Type of the notification event. It MUST be a case sensitive string whose value is either `credential_accepted`, `credential_failure` or `credential_deleted`. `credential_accepted` is to be used when the Credential was successfully stored in the Wallet, with or without user action. `credential_deleted` is to be used when the unsuccessful Credential issuance was caused by a user action. In all other unsuccessful cases, `credential_failure` is to be used.
* `error_description`: OPTIONAL. Human-readable ASCII [@!USASCII] text providing additional information, used to assist the Credential Issuer developer in understanding the error that occurred. Values for the `error_description` parameter MUST NOT include characters outside the set `%x20-21 / %x23-5B / %x5D-7E`.

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
  "event": "credential_deleted",
  "error_description": "..."
}
```

## Successful Notification Response

When the Credential Issuer has successfully received the Notification Request from the Wallet, it MUST respond with the HTTP status code 2xx. The usage of the HTTP status code 204 (No Content) is RECOMMENDED.

Below is a non-normative example of response to a successful Notification Request:

```
HTTP/1.1 204 No Content
```

## Notification Error Response

If the Notification Request does not contain an Access Token or contains an invalid Access Token, the Notification Endpoint returns an Authorization Error Response such as defined in section 3 of [@!RFC6750].

When the `notification_id` value is invalid, the HTTP response MUST use the HTTP status code 400 (Bad Request) and set the content type to `application/json` with the following parameters in the JSON-encoded response body:

* `error`: REQUIRED. A key at the top level of the object, the value of which SHOULD be the following ASCII [@!USASCII] error code:
  * `invalid_notification_id`: The `notification_id` in the Notification Request was invalid.
  * `invalid_notification_request`: The Notification Request is missing a required parameter, includes an unsupported parameter or parameter value, repeats the same parameter, or is otherwise malformed.

It is at the discretion of the Issuer to decide how to proceed after returning an error response.

The following is a non-normative example of an Notification Error Response where an invalid `notification_id` value was used:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store
{
   "error": "invalid_notification_id"
}
```

# Metadata

## Client Metadata {#client_metadata}

This specification defines the following new Client Metadata parameter in addition to [@!RFC7591] for Wallets acting as OAuth 2.0 Client:

* `credential_offer_endpoint`: OPTIONAL. Credential Offer Endpoint of a Wallet.

### Client Metadata Retrieval {#client_metadata_retrieval}

How to obtain Client Metadata is out of scope of this specification. The profiles of this specification may define static Client Metadata values.

If the Credential Issuer is unable to perform discovery of the Wallet's Credential Offer Endpoint, the following claimed URL is used: `openid-credential-offer://`.

## Credential Issuer Metadata {#credential_issuer_metadata}

The Credential Issuer Metadata contains information on the Credential Issuer's technical capabilities, supported Credentials and (internationalized) display information.

### Credential Issuer Identifier {#credential_issuer_identifier}

A Credential Issuer is identified by a case sensitive URL using the `https` scheme that contains scheme, host and, optionally, port number and path components, but no query or fragment components. 

### Credential Issuer Metadata Retrieval {#credential_issuer_wellknown}

The Credential Issuer's configuration can be retrieved using the Credential Issuer Identifier.

Credential Issuers publishing metadata MUST make a JSON document available at the path formed by concatenating the string `/.well-known/openid-credential-issuer` to the Credential Issuer Identifier. If the Credential Issuer value contains a path component, any terminating `/` MUST be removed before appending `/.well-known/openid-credential-issuer`.

Communication with the Credential Issuer Metadata Endpoint MUST utilize TLS.

To fetch the Credential Issuer Metadata, a requester MUST send a HTTP request using the GET method and the path formed following the steps above. The URL SHOULD NOT contain any query parameters. The Credential Issuer MUST return a JSON document compliant with this specification using the `application/json` media type and a HTTP Status Code 200.

The Wallet is RECOMMENDED to send an `Accept-Language` Header in the HTTP GET request to indicate the language(s) preferred for display. It is up to the Credential Issuer whether to

* send a subset the metadata containing internationalized display data for one or all of the requested languages and indicate returned languages using the HTTP `Content-Language` Header.
* ignore the `Accept-Language` Header and send all supported languages or any chosen default subset.

The language(s) in HTTP `Accept-Language` and `Content-Language` Headers MUST use the values defined in [@!RFC3066]. 

Below is a non-normative example of a Credential Issuer Metadata request:

```
GET /.well-known/openid-credential-issuer HTTP/1.1
Host: server.example.com
Accept-Language: fr-ch, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5
```

### Credential Issuer Metadata Parameters {#credential-issuer-parameters}

This specification defines the following Credential Issuer Metadata:

* `credential_issuer`: REQUIRED. The Credential Issuer's identifier, as defined in (#credential_issuer_identifier).
* `authorization_servers`: OPTIONAL. Array of strings, where each string is an identifier of the OAuth 2.0 Authorization Server (as defined in [@!RFC8414]) the Credential Issuer relies on for authorization. If this parameter is omitted, the entity providing the Credential Issuer is also acting as the AS, i.e., the Credential Issuer's identifier is used as the OAuth 2.0 Issuer value to obtain the Authorization Server metadata as per [@!RFC8414]. When there are multiple entries in the array, the Wallet may be able to determine which AS to use by querying the metadata; for example, by examining the `grant_types_supported` values, the Wallet can filter the server to use based on the grant type it plans to use. When the Wallet is using `authorization_server` parameter in the Credential Offer as a hint to determine which AS to use out of multiple, the Wallet MUST NOT proceed with the flow if the `authorization_server` Credential Offer parameter value does not match any of the entries in the `authorization_servers` array.
* `credential_endpoint`: REQUIRED. URL of the Credential Issuer's Credential Endpoint as defined in (#credential_request). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components.
* `batch_credential_endpoint`: OPTIONAL. URL of the Credential Issuer's Batch Credential Endpoint as defined in (#batch-credential-endpoint). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Batch Credential Endpoint.
* `deferred_credential_endpoint`: OPTIONAL. URL of the Credential Issuer's Deferred Credential Endpoint as defined in (#deferred-credential-issuance). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Deferred Credential Endpoint.
* `notification_endpoint`: OPTIONAL. URL of the Credential Issuer's Notification Endpoint as defined in (#notification_endpoint). This URL MUST use the `https` scheme and MAY contain port, path, and query parameter components. If omitted, the Credential Issuer does not support the Notification Endpoint.
* `credential_response_encryption_alg_values_supported`: OPTIONAL. Array containing a list of the JWE [@!RFC7516] encryption algorithms (`alg` values) [@!RFC7518] supported by the Credential and/or Batch Credential Endpoint to encode the Credential or Batch Credential Response in a JWT [@!RFC7519].
* `credential_response_encryption_enc_values_supported`: OPTIONAL. Array containing a list of the JWE [@!RFC7516] encryption algorithms (`enc` values) [@!RFC7518] supported by the Credential and/or Batch Credential Endpoint to encode the Credential or Batch Credential Response in a JWT [@!RFC7519].
* `require_credential_response_encryption`: OPTIONAL. Boolean value specifying whether the Credential Issuer requires additional encryption on top of TLS for the Credential Response and expects encryption parameters to be present in the Credential Request and/or Batch Credential Request, with `true` indicating support. When the value is `true`, `credential_response_encryption_alg_values_supported` parameter MUST also be provided.  If omitted, the default value is `false`.
* `credential_identifiers_supported`: OPTIONAL. Boolean value specifying whether the Credential Issuer supports returning `credential_identifiers` parameter in the `authorization_details` Token Response parameter, with `true` indicating support. If omitted, the default value is `false`.
* `display`: OPTIONAL. Array of objects, where each object contains display properties of a Credential Issuer for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
  * `name`: OPTIONAL. String value of a display name for the Credential Issuer.
  * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `credential_configurations_supported`: REQUIRED. Object that describes specifics of the Credential that the Credential Issuer supports issuance of. This object contains a list of name/value pairs, where each name is a unique identifier of the supported Credential being described. This identifier is used in the Credential Offer as defined in (#credential_offer_parameters) to communicate to the Wallet which Credential is being offered. The value is an object that contains metadata about specific Credential and contains the following parameters defined by this specification:
  * `format`: REQUIRED. A JSON string identifying the format of this Credential, i.e., `jwt_vc_json` or `ldp_vc`. Depending on the format value, the object contains further elements defining the type and (optionally) particular claims the Credential MAY contain and information about how to display the Credential. (#format_profiles) defines Credential Format Profiles introduced by this specification.
  * `scope`: OPTIONAL. A JSON string identifying the scope value that this Credential Issuer supports for this particular Credential. The value can be the same accross multiple `credential_configurations_supported` objects. The Authorization Server MUST be able to uniquely identify the Credential Issuer based on the scope value. The Wallet can use this value in the Authorization Request as defined in (#credential-request-using-type-specific-scope). Scope values in this Credential Issuer metadata MAY duplicate those in the `scopes_supported` parameter of the Authorization Server.
  * `cryptographic_binding_methods_supported`: OPTIONAL. Array of case sensitive strings that identify how the Credential is bound to the identifier of the End-User who possesses the Credential as defined in (#credential-binding). Support for keys in JWK format [@!RFC7517] is indicated by the value `jwk`. Support for keys expressed as a COSE Key object [@!RFC8152] (for example, used in [@!ISO.18013-5]) is indicated by the value `cose_key`. When Cryptographic Binding Method is a DID, valid values MUST be a `did:` prefix followed by a method-name using a syntax as defined in Section 3.1 of [@!DID-Core], but without a `:`and method-specific-id. For example, support for the DID method with a method-name "example" would be represented by `did:example`. Support for all DID methods listed in Section 13 of [@DID_Specification_Registries] is indicated by sending a DID without any method-name.
  * `cryptographic_suites_supported`: OPTIONAL. Array of case sensitive strings that identify the cryptographic suites that are supported for the `cryptographic_binding_methods_supported`. Cryptographic algorithms for Credentials in `jwt_vc` format should use algorithm names defined in [IANA JOSE Algorithms Registry](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms). Cryptographic algorithms for Credentials in `ldp_vc` format should use signature suites names defined in [Linked Data Cryptographic Suite Registry](https://w3c-ccg.github.io/ld-cryptosuite-registry/).
  * `proof_types`: OPTIONAL. Array of case sensitive strings, each representing a `proof_type` that the Credential Issuer supports as defined in (#credential_request), one of which MUST be used in the Credential Request. If this array is non-empty and present, the Credential Issuer requires proof of possession of the cryptographic key material. If the parameter is omitted or the array is empty, the Credential Issuer does not require proof of possession of the cryptographic key material.
  * `display`: OPTIONAL. Array of objects, where each object contains the display properties of the supported Credential for a certain language. Below is a non-exhaustive list of parameters that MAY be included.
      * `name`: REQUIRED. String value of a display name for the Credential.
      * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. Multiple `display` objects MAY be included for separate languages. There MUST be only one object for each language identifier.
      * `logo`: OPTIONAL. Object with information about the logo of the Credential with a following non-exhaustive list of parameters that MAY be included:
          * `url`: OPTIONAL. URL where the Wallet can obtain a logo of the Credential from the Credential Issuer.
          * `alt_text`: OPTIONAL. String value of an alternative text of a logo image.
      * `description`: OPTIONAL. String value of a description of the Credential.
      * `background_color`: OPTIONAL. String value of a background color of the Credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].
      * `text_color`: OPTIONAL. String value of a text color of the Credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].

Note: It can be challenging for a Credential Issuer that accepts tokens from multiple Authorization Servers to introspect an Access Token to check the validity and determine the permissions granted. Some ways to achieve this are relying on Authorization Servers that use [@!RFC9068] or by the Credential Issuer understanding the proprietary Access Token structures of the Authorization Servers.

Depending on the Credential format, additional parameters might be present in the `credential_configurations_supported` object values, such as information about claims in the Credential. For Credential format specific claims, see "Credential Issuer Metadata" subsections in (#format_profiles).

The AS MUST be able to determine from the Issuer metadata what claims are disclosed with the requested Credentials to be able to render a meaningful End-User consent.

The following is a non-normative example of a Credential Issuer metadata.

<{{examples/credential_issuer_metadata_jwt_vc_json.json}}

Note: The Client MAY use other mechanisms to obtain information about the Verifiable Credentials that a Credential Issuer can issue.

## OAuth 2.0 Authorization Server Metadata

This specification also defines a new OAuth 2.0 Authorization Server metadata [@!RFC8414] parameter to publish whether the AS that the Credential Issuer relies on for authorization supports anonymous Token Requests with the Pre-Authorized Grant Type. It is defined as follows:

* `pre-authorized_grant_anonymous_access_supported`: OPTIONAL. A boolean indicating whether the Credential Issuer accepts a Token Request with a Pre-Authorized Code but without a `client_id`. The default is `false`. 

# Security Considerations {#security-considerations}

## Trust between Wallet and Issuer

Credential Issuers often want to know what Wallet they are issuing Credentials to and how private keys are managed for the following reasons:

* The Credential Issuer MAY want to ensure that private keys are properly protected from exfiltration and replay to prevent an adversary from impersonating the legitimate Credential Holder by presenting their Credentials.
* The Credential Issuer MAY also want to ensure that the Wallet managing the Credentials adheres to certain policies and, potentially, was audited and approved under a certain regulatory and/or commercial scheme.

The following mechanisms in concert can be utilized to fulfill those objectives:

**Key attestation** is a mechanism where the device or security element in a device asserts the key management policy to the application creating and using this key. The Android Operating System, for example, provides apps with a certificate including a certificate chain asserting that a particular key is managed, for example, by a hardware security module [ref]. The Wallet can provide this data along with the proof of possession in the Credential Request (see (#credential_request) for an example) to allow the Credential Issuer to validate the key management policy. This requires the Credential Issuer to rely on the trust anchor of the certificate chain and the respective key management policy. Another variant of this concept is the use of a Qualified Electronic Signature as defined by the eIDAS regulation [ref]. This signature will not reveal the properties of the associated private key to the Credential Issuer. However, as one example, due to the regulatory regime of eIDAS, the Credential Issuer can deduce that the signing service manages the private keys according to this regime and fulfills very high security requirements. As another example, FIDO2 allows RPs to obtain an attestation along with the public key from a FIDO authenticator. That implicitly asserts the key management policy since the assertion is bound to a certain authenticator model and its key management capabilities.

**App Attestation**: Key attestation, however, does not establish trust in the application storing the Credential and producing presentation of that Credential. App attestation as provided by mobile operating systems, e.g., iOS's DeviceCheck or Android's SafetyNet, allows a server system to ensure it is communicating to a legitimate instance of its genuine app. Those mechanisms can be utilized to validate the internal integrity of the Wallet (as a whole).

**Device Attestation**: Device Attestation attests the health of the device on which the Wallet is running, as a whole. It helps prevent compromises such as a malicious third-party application tampering with a Wallet that manages keys and Credentials, which cannot be captured only by obtaining app attestation of a Wallet.

**Client Authentication** allows a Wallet to authenticate with a Credential Issuer. To securely authenticate, the Wallet needs to utilize a backend component managing the key material and processing the secure communication with the Credential Issuer. The Credential Issuer MAY establish trust with the Wallet based on its own auditing or a trusted third-party attestation of the conformance of the Wallet to a certain policy.

Directly using key, app and/or device attestations to prove certain capabilities towards a Credential Issuer is an obvious option. However, this at least requires platform mechanisms that issue signed assertions that third parties can evaluate, which is not always the case (e.g., iOS's DeviceCheck). Also, such an approach creates dependencies on platform specific mechanisms, trust anchors, and platform specific identifiers (e.g., Android `apkPackageName`) and it reveals information about the internal design of a Wallet. Implementers should take these consequences into account.

The approach recommended by this specification is that the Credential Issuer relies on the OAuth 2.0 Client Authentication to establish trust in the Wallet and leaves it to the Wallet to ensure its internal integrity using app and key attestation (if required). This establishes a clean separation between the different components and a uniform interface irrespective of the Wallet's architecture (e.g., native vs. Web Wallet). Client Authentication can be performed with assertions registered with the Credential Issuer or with assertions issued to the Wallet by a third party the Credential Issuer trusts for the purpose of Client Authentication.

## Credential Offer {#credential-offer-security}

The Wallet MUST consider the parameter values in the Credential Offer as not trustworthy since the origin is not authenticated and the message integrity is not protected. The Wallet MUST apply the same checks on the Credential Issuer that it would apply when the flow is started from the Wallet itself since the Credential Issuer is not trustworthy just because it sent the Credential Offer. An attacker might attempt to use a Credential Offer to conduct a phishing or injection attack. 

The Wallet MUST NOT accept Credentials just because this mechanism was used. All protocol steps defined in this specification MUST be performed in the same way as if the Wallet would have started the flow. 

The Credential Issuer MUST ensure the release of any privacy-sensitive data in Credential Offer is legally based.

## Pre-Authorized Code Flow {#security_considerations_pre-authz-code}

### Replay Prevention

The Pre-Authorized Code Flow is vulnerable to the replay of the Pre-Authorized Code, because by design, it is not bound to a certain device (as the Authorization Code Flow does with PKCE). This means an attacker can replay at another device the Pre-Authorized Code meant for a victim, e.g., the attacker can scan the QR code while it is displayed on the victim's screen, and thereby get access to the Credential. Such replay attacks must be prevented using other means. The design facilitates the following options:

* Transaction Code: the Credential Issuer might set up a Transaction Code with the End-User (e.g., via text message or email) that needs to be presented in the Token Request.
* Callback to device where the transaction originated: upon receiving the Token Request, the Credential Issuer asks the End-User to confirm the originating device (device that displayed the QR code) that the Credential Issuer MAY proceed with the Credential issuance process. While the Credential Issuer reaches out to the End-User on the other device to get confirmation, the Credential Issuer's Authorization Server returns an error code `authorization_pending` or `slow_down` to the Wallet as described in (#token_error_response). The Wallet is required to call the Token Endpoint again to obtain the Access Token. If the End-User does not confirm, the Token Request is returned with the `access_denied` error code. This flow gives the End-User on the originating device more control over the issuance process.

### Transaction Code Phishing

An attacker might leverage the Credential issuance process and the End-User's trust into the Wallet to phish Transaction Codes sent out by a different service that grant the attacker access to services other than Credential issuance. The attacker could set up a Credential Issuer site and in parallel to the issuance request trigger transmission of a Transaction Code to the End-User's phone from a service other than Credential issuance, e.g., from a payment service. The End-User would then be asked to enter this Transaction Code into the Wallet and since the Wallet sends this Transaction Code to the Token Endpoint of the Credential Issuer (the attacker), the attacker would get access to the Transaction Code, and access to that other service.

In order to cope with that issue, the Wallet is RECOMMENDED to interact with trusted Credential Issuers only. In that case, the Wallet would not process a Credential Offer with an untrusted issuer URL. The Wallet MAY also show the End-User the endpoint of the Credential Issuer it will be sending the Transaction Code to and ask the End-User for confirmation.

## Credential Lifecycle Management 

The Credential Issuer is supposed to be responsible for the lifecycle of its Credentials. This means the Credential Issuer will invalidate Credentials when it deems appropriate, e.g., if it detects fraudulent behavior.

The Wallet is supposed to detect signs of fraudulent behavior related to the Credential management in the Wallet (e.g., device rooting) and to act upon such signals. Options include Credential revocation at the Credential Issuer and/or invalidation of the key material used to cryptographically bind the Credential to the identifier of the End-User possessing that Credential.

## Key Proof replay {#key-proof-replay}

If an adversary is able to get hold of a key proof defined in (#proof_types), the adversary could get a Credential issued that is bound to a key pair controlled by the victim.

Note: For the attacker to be able to present to the Verifier a Credential bound to a replayed Key Proof, the attacker also needs to obtain the victim's private key. To limit this, servers are RECOMMENDED to check how the Wallet protects the private keys, using mechanisms such as Attestation-Based Client Authentication defined in [@!I-D.ietf-oauth-attestation-based-client-auth].

`nonce` parameter is the primary countermeasure against key proof replay. To further narrow down the attack vector, the Credential Issuer SHOULD bind a unique `nonce` parameter to the respective Access Token.

Note: To accommodate for clock offsets, the Credential Issuer server MAY accept proofs that carry an `iat` time in the reasonably near future (on the order of seconds or minutes). Because clock skews between servers and Clients may be large, servers MAY limit key proof lifetimes by using server-provided nonce values containing the time at the server rather than comparing the client-supplied `iat` time to the time at the server. Nonces created in this way yield the same result even in the face of arbitrarily large clock skews.

Server-provided nonces are an effective means for further reducing the chances for successful key proof replay by an attacker. A Wallet can keep using a certain nonce until the Credential Issuer provides a fresh nonce. This way the Credential Issuer determines how often a certain nonce can be used. Servers MUST have a clear policy whether the same key proof can be presented multiple times and for how long, or whether each Credential Request MUST have a fresh key proof.

##  TLS Requirements

Implementations MUST follow [@!BCP195].

Whenever TLS is used, a TLS server certificate check MUST be performed, per [@!RFC6125].

# Implementation Considerations

## Claim-based Binding of the Credential to the End-User possessing the Credential {#claim-based-binding}

Credentials not cryptographically bound to the identifier of the End-User possessing it (see (#credential-binding)), should be bound to the End-User possessing the Credential, based on the claims included in that Credential.

In claim-based binding, no cryptographic binding material is provided. Instead, the issued Credential includes End-User claims that can be used by the Verifier to verify possession of the Credential by requesting presentation of existing forms of physical or digital identification that includes the same claims (e.g., a driving license or other ID cards in person, or an online ID verification service).

## Binding of the Credential without Cryptographic Binding or Claim-based Binding {#no-binding}

Some Credential Issuers might choose to issue bearer Credentials without either cryptographic binding or claim-based binding, because they are meant to be presented without proof of possession.

One such use case is low assurance Credentials, such as coupons or tickets.

Another use case is when the Credential Issuer uses cryptographic schemes that can provide binding to the End-User possessing that Credential without explicit cryptographic material being supplied by the application used by that End-User. For example, in the case of the BBS Signature Scheme, the issued Credential itself is a secret and only derivation of a Credential is presented to the Verifier. Effectively, Credential is bound to the Credential Issuer's signature on the Credential, which becomes a shared secret transferred from the Credential Issuer to the End-User.

## Multiple Accesses to the Credential Endpoint

The Credential Endpoint can be accessed multiple times by a Wallet using the same Access Token, even for the same Credential. The Credential Issuer determines if the subsequent successful requests will return the same or an updated Credential, such as having a new expiration time or using the most current End-User claims.

The Credential Issuer MAY also decide to no longer accept the Access Token and a re-authentication or Token Refresh (see [@!RFC6749], section 6) MAY be required at the Credential Issuer's discretion. The policies between the Credential Endpoint and the Authorization Server that MAY change the behavior of what is returned with a new Access Token are beyond the scope of this specification (see [@!RFC6749, section 7]).

The action leading to the Wallet performing another Credential Request can also be triggered by a background process, or by the Credential Issuer using an out-of-band mechanism (SMS, email, etc.) to inform the End-User.

## Relationship between the Credential Issuer Identifier in the metadata and the Issuer Identifier in the Issued Credential

Credential Issuer Identifier is always a URL using the `https` scheme as defined in (#credential_issuer_identifier). Depending on the Credential format, the issuer identifier in the issued Credential may not be a URL using the `https` scheme. Some other forms that it can take are a DID included in the `issuer` property in a [@VC_DATA] format, or the `Subject` value of the document signer certificate included in the `x5chain` element in a [@ISO.18013-5] format.

When the Issuer identifier in the issued Credential is a DID, below is a non-exhaustive list of mechanisms the Credential Issuer MAY use to bind to the Credential Issuer Identifier:

1. Use the [@DIF.Well-Known_DID] Specification to provide binding between a DID and a certain domain.
1. If the Issuer identifier in the issued Credential is an object, add to the object a `credential_issuer` claim, as defined in (#credential_issuer_identifier).

The Wallet MAY check the binding between the Credential Issuer Identifier and the Issuer identifier in the issued Credential.

## Refreshing Issued Credentials

After a Verifiable Credential has been issued to the Holder, claim values about the subject of a Credential, or a signature on the Credential may need to be updated. There are two possible mechanisms to do so.

First, the Wallet may receive an updated version of a Credential from a Credential Endpoint or a Batch Credential Endpoint using a valid Access Token. This does not involve interaction with the End-User. If the Credential Issuer issued a Refresh Token to the Wallet, the Wallet would have to obtain a fresh Access Token by making a request to the Token Endpoint as defined in Section 6 of [@!RFC6749].

Second, the Credential Issuer can reissue the Credential by starting the issuance process from the beginning. This would involve interaction with the End-User. A Credential needs to be reissued if the Wallet does not have a valid Access Token or a valid Refresh Token. With this approach, when a new Credential is issued, the Wallet might need to check if it already has a Credential of the same type and, if necessary, delete the old Credential. Otherwise, the Wallet might end up with more than one Credential of the same type, without knowing which one is the latest.

Credential Refresh can be initiated by the Wallet independently from the Credential Issuer, or the Credential Issuer can send a signal to the Wallet asking it to request Credential refresh. How the Credential Issuer sends such a signal is out of scope for this specification.

It is up to the Credential Issuer whether to update both the signature and the claim values, or only the signature.

## Protecting the Access Token

Access Tokens represent End-User authorization and consent to issue certain Credential(s). Long-lived Access Tokens giving access to Credentials MUST not be issued unless sender-constrained. Access Tokens with lifetimes longer than 5 minutes are, in general, considered long lived.

To sender-constrain Access Tokens, see the recommendations in Section 4.10.1 in [@!I-D.ietf-oauth-security-topics]. If Bearer Access Tokens are stored by the Wallet, they MUST be stored in a secure manner, for example, encrypted using a key stored in a protected key store.

# Privacy Considerations

TBD

{backmatter}

<reference anchor="DID-Core" target="https://www.w3.org/TR/2021/PR-did-core-20210803/">
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
            <organization>Evernym</organization>
        </author>
        <date day="3" month="Aug" year="2021"/>
        </front>
</reference>

<reference anchor="VC_DATA" target="https://www.w3.org/TR/vc-data-model">
  <front>
    <title>Verifiable Credentials Data Model 1.0</title>
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
    <author fullname="David Chadwick">
      <organization>University of Kent</organization>
    </author>
   <date day="19" month="Nov" year="2019"/>
  </front>
</reference>

<reference anchor="VC_DATA_2.0" target="https://www.w3.org/TR/vc-data-model-2.0">
  <front>
    <title>Verifiable Credentials Data Model 2.0</title>
    <author fullname="Manu Sporny">
      <organization>Digital Bazaar</organization>
    </author>
    <author fullname="Orie Steele">
      <organization>Transmute</organization>
    </author>
    <author fullname="Oliver Terbu">
      <organization>Spruce Systems, Inc.</organization>
    </author>
    <author fullname="Grant Noble">
      <organization>ConsenSys</organization>
    </author>
    <author fullname="Gabe Cohen">
      <organization>Block</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>independent</organization>
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
      <organization>MATTR</organization>
    </author>
    <author fullname="David Chadwick">
      <organization>University of Kent</organization>
    </author>
   <date day="15" month="Aug" year="2023"/>
  </front>
</reference>

<reference anchor="Data_Integrity" target="https://w3c.github.io/vc-data-integrity/">
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
   <date day="31" month="Aug" year="2023"/>
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
      <organization>Microsoft</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Self-Issued Consulting</organization>
    </author>
    <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
      <organization>sprind.org</organization>
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
          <date year="2022"/>
        </front>
</reference>

<reference anchor="OpenID.Core" target="http://openid.net/specs/openid-connect-core-1_0.html">
  <front>
    <title>OpenID Connect Core 1.0 incorporating errata set 1</title>
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
   <date day="8" month="Nov" year="2014"/>
  </front>
</reference>

<reference anchor="DIF.PresentationExchange" target="https://identity.foundation/presentation-exchange/spec/v1.0.0/">
        <front>
          <title>Presentation Exchange v1.0.0</title>
      <author fullname="Daniel Buchner">
            <organization>Microsoft</organization>
          </author>
          <author fullname="Brent Zundel">
            <organization>Evernym</organization>
          </author>
          <author fullname="Martin Riedel">
            <organization>Consensys Mesh</organization>
          </author>
         <date month="Feb" year="2021"/>
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
          <date year="2021"/>
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
          <organization>sprind.org</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>Microsoft</organization>
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
          <author fullname="Michael Prorock">
            <organization>mesur.io</organization>
          </author>
         <date month="Aug" year="2022"/>
        </front>
</reference>

<reference anchor="OpenID.Federation" target="https://openid.net/specs/openid-connect-federation-1_0.html">
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

# IANA Considerations

## Sub-Namespace Registration

This specification registers the following URN in the IANA "OAuth URI" registry [@!IANA.OAuth.Parameters] established by [@!RFC6755].

* URN: urn:ietf:params:oauth:grant-type:pre-authorized_code
* Common Name: Pre-Authorized Code
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token_request) of this spedification

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
* Reference: (#token_request) of this specification

* Parameter Name: tx_code
* Parameter Usage Location: token request
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token_request) of this specification

* Parameter Name: c_nonce
* Parameter Usage Location: token response
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-response) of this specification

* Parameter Name: c_nonce_expires_in
* Parameter Usage Location: token response
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#token-response) of this specification

## OAuth Dynamic Client Registration Metadata Registry

This specification registers the following client metadata name in the IANA "OAuth Dynamic Client Registration Metadata" registry [@!IANA.OAuth.Parameters] established by [@!RFC7591].

* Client Metadata Name: credential_offer_endpoint
* Client Metadata Description: Credential Offer Endpoint
* Change Controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Reference: (#credential_offer_endpoint) of this specification


## Well-Known URI Registry

This specification registers the following well-known URI in the IANA "Well-Known URI" registry established by [@!RFC5785].

* URI suffix: openid-credential-issuer
* Change controller: OpenID Foundation Digital Credentials Protocols Working Group - openid-specs-digital-credentials-protocols@lists.openid.net
* Specification document: (#credential_issuer_wellknown) of this document
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

* Type name: `application`
* Subtype name: `openid4vci-proof+cwt`
* Required parameters: n/a
* Optional parameters: n/a
* Encoding considerations: Binary CBOR, as specified in [@!RFC9052]
* Security considerations: See the Security Considerations in [@!RFC8392].
* Interoperability considerations: n/a
* Published specification: (#cwt-proof-type) of this specification
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

# Acknowledgements {#Acknowledgements}

We would like to thank Paul Bastian, Vittorio Bertocci, Christian Bormann, John Bradley, Brian Campbell, Gabe Cohen, David Chadwick, Andrii Deinega, Giuseppe De Marco, Mark Dobrinic, Daniel Fett, Pedro Felix, George Fletcher, Timo Glasta, Mark Haine, Fabian Hauck, Roland Hedberg, Joseph Heenan, Alen Horvat, Andrew Hughes, Jacob Ideskog, Edmund Jay, Michael B. Jones, Tom Jones, Judith Kahrer, Takahiko Kawasaki, Niels Klomp, Ronald Koenig, Markus Kreusch, Adam Lemmon, Daniel McGrogan, Jeremie Miller, Kenichi Nakamura, Rolson Quadras, Nat Sakimura, Oliver Terbu, Arjen van Veen, David Waite, Jacob Ward for their valuable feedback and contributions to this specification.

# Notices

Copyright (c) 2023 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that MAY cover technology that MAY be required to practice this specification.

# Use Cases

This is a non-exhaustive list of sample use cases.

## Credential Offer - Same-Device {#use-case-1}

While browsing the university's home page, the End-User finds a link "request your digital diploma". The End-User clicks on this link and is being redirected to a digital Wallet. The Wallet notifies the End-User that a Credential Issuer offered to issue a diploma Credential. User confirms this inquiry and is taken to the university's Credential issuance service's End-User experience. Upon successful authentication at the university and consent to the issuance of a digital diploma, the End-User is redirected back to the Wallet. Here, the End-User can verify the successful creation of the digital diploma.

## Credential Offer - Cross-Device (with Information Pre-Submitted by the End-User) {#use-case-2}

The End-User is starting a job at a new employer. The employer requests the End-User to upload specific documents to the employee portal. After a few days, the End-User receives an email from the employer indicating that the employee Credential is ready to be claimed and provides instructions to scan a presented QR code for its retrieval. The End-User scans the QR code with the smartphone, which opens the Wallet. Meanwhile, the End-User has received a text message with a Transaction Code to the smartphone. After entering the Transaction Code in the Wallet for security reasons, the End-User confirms the Credential issuance, and receives the Credential into the Wallet.

## Credential Offer - Cross-Device & Deferred {#use-case-3}

The End-User intends to acquire a digital criminal record. This involves a visit to the local administration's office to request the official criminal record be issued as a digital Credential. After presenting the ID document, the End-User is prompted to scan a QR code using the Wallet and is informed that the issuance of the Credential will require some time, due to necessary background checks by the authority.

While using the Wallet, the End-User notices an indication that the issuance of the digital record is in progress. After a few days, the End-User receives a notification from the Wallet indicating that the requested Credential was successfully issued. Upon opening the Wallet, the End-User is queried about the download of the Credential. After confirmation, the Wallet fetches and saves the new Credential.

## Wallet Initiated Issuance during Presentation {#use-case-4}

An End-User comes across a verifier app that is requesting the End-User to present a Credential, e.g., a driving license. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet selects a Credential Issuer capable of issuing the missing Credential and, upon End-User consent, sends the End-User to the Credential Issuer's End-User experience (Web site or app). Once authenticated and consent is provided for the issuance of the Credential into the Wallet, the End-User is redirected back to the Wallet. The Wallet informs the End-User that Credential was successfully issued into the Wallet and is ready to be presented to the verifier app that originally requested presentation of that Credential.

## Wallet Initiated Issuance during Presentation (Requires Presentation of Additional Credentials During Issuance) {#use-case-5}

An End-User comes across a verifier app that is requesting the End-User to present a Credential, e.g., a university diploma. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet then offers the End-User a list of Credential Issuers, which might be based on a Credential Issuer list curated by the Wallet provider. The End-User selects the university of graduation and is subsequently redirected to the corresponding university's website or app.

The End-User logs into the university, which identifies that the corresponding End-User account is not yet verified. Among various identification options, the End-User opts to present a Credential from the Wallet. The End-User is redirected back to the Wallet to consent to present the requested Credential(s) to the university. Following this, the End-User is redirected back to the university End-User experience. Based on the presented Credential, the university finalizes the End-User verification, retrieves End-User data from its database, and proposes to issue a diploma as a Verifiable Credential.

Upon providing consent, the End-User is sent back to the Wallet. The Wallet informs the End-User Credential was successfully issued into the Wallet and is ready to be presented to the verifier app that originally requested presentation of that Credential.

## Wallet Initiated Issuance after Installation {#use-case-6}

The End-User installs a new Wallet and executes it. The Wallet offers the End-User a selection of the Credentials that the End-User may obtain from a Credential Issuer, e.g. a national identity credential, a mobile driving license or a public transport ticket. The corresponding Credential Issuers (and their URLs) are pre-configured by the Wallet or follow some discovery processes that are out of scope for this specification. By clicking on one these options corresponding to the Credentials available for the issuance, the Issuance process starts with the flows supported by the Credential Issuer (Pre-Authorized Code flow or Auth Code flow).

Wallet Providers may also provide a market place where Issuers can register to be found for Wallet-initiated flows.

# Credential Format Profiles {#format_profiles}

This specification defines several extension points to accommodate the differences across Credential formats. Sets of Credential format specific parameters or claims referred to as Credential format Identifiers are identified by the Credential format identifier and used at each extension point.

This section defines Credential Format Profiles for a few of the commonly used Credential formats. Other specifications or deployments can define their own Credential Format Profiles.

## W3C Verifiable Credentials

Sections 6.1 and 6.2 of [@VC_DATA] define how Verifiable Credentials MAY or MAY NOT use JSON-LD. As acknowledged in Section 4.1 of [@VC_DATA], implementations can behave differently regarding processing of the `@context` property whether JSON-LD is used or not.

This specification therefore differentiates the following three Credential formats for W3C Verifiable Credentials:

* VC signed as a JWT, not using JSON-LD (`jwt_vc_json`)
* VC signed as a JWT, using JSON-LD (`jwt_vc_json-ld`)
* VC secured using Data Integrity, using JSON-LD, with proof suite requiring Linked Data canonicalization (`ldp_vc`)

Note: VCs secured using Data Integrity MAY NOT necessarily use JSON-LD and MAY NOT necessarily use proof suites requiring Linked Data canonicalization. Credential Format Profiles for them may be defined in the future versions of this specification.

Distinct Credential format identifiers, extension parameters/claims, and processing rules are defined for each of the above-mentioned Credential formats.

It is on purpose that the Credential Offer does not contain `credentialSubject` property, while Authorization Details and Credential Request do. This is because this property is meant to be used by the Wallet to specify which claims it is requesting to be issued out of all the claims the Credential Issuer is capable of issuing for this particular Credential (data minimization), while Credential Offer is a mere "invitation" from the Credential Issuer to the Wallet to start the issuance flow.

### VC Signed as a JWT, Not Using JSON-LD {#vc-jwt}

#### Format Identifier

The Credential format identifier is `jwt_vc_json`.

When the `format` value is `jwt_vc_json`, entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules.

#### Credential Issuer Metadata {#server_metadata_jwt_vc_json}

The following additional Credential Issuer metadata are defined for this Credential format to be added to the `credential_configurations_supported` parameter in addition to those defined in (#credential-issuer-parameters).

* `credential_definition`: REQUIRED. Object containing the detailed description of the Credential type. It consists at least of the following two sub claims:
  * `type`: REQUIRED. Array designating the types a certain Credential type supports according to [@VC_DATA], Section 4.3.
  * `credentialSubject`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. To express the specifics about the claim, the most deeply nested value MAY be an object that includes a following non-exhaustive list of parameters defined by this specification:
      * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the Credential Issuer will always include this claim in the issued Credential. If set to `false`, the claim is not included in the issued Credential if the wallet did not request the inclusion of the claim, and/or if the Credential Issuer chose to not include the claim. If the `mandatory` parameter is omitted, the default should be assumed to be `false`.
      * `value_type`: OPTIONAL. String value determining the type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
      * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
          * `name`: OPTIONAL. String value of a display name for the claim.
          * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. Array of the claim name values that lists them in the order they should be displayed by the Wallet.

The following is a non-normative example of an object comprising `credential_configurations_supported` parameter of Credential format `jwt_vc_json`:

<{{examples/credential_metadata_jwt_vc_json.json}}

#### Authorization Details {#authorization_jwt_vc_json}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.

* `credential_definition`: OPTIONAL. Object containing a detailed description of the Credential consisting of the following sub claim:
  * `credentialSubject`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. The most deeply nested value MUST be an empty object. This object indicates the claims the Wallet would like to turn up in the Credential to be issued.

Note that the `type` is referenced in the `credential_configurations_supported` object in the Credential Issuer metadata.

The following is a non-normative example of an authorization details object with Credential format `jwt_vc_json`:

<{{examples/authorization_details_jwt_vc_json.json}}

#### Credential Request

The following additional parameters are defined for Credential Requests and this Credential format.  

* `credential_definition`: REQUIRED when the `credential_identifier` was not present in the Credential Request. MUST NOT be used otherwise. Object containing the detailed description of the Credential type. It consists at least of the following sub claims:
  * `type`: REQUIRED. Array as defined in (#server_metadata_jwt_vc_json). The credential issued by the Credential Issuer MUST at least contain the values listed in this claim.
  * `credentialSubject`: OPTIONAL. Object as defined in (#authorization_jwt_vc_json).

The following is a non-normative example of a Credential Request with Credential format `jwt_vc_json`:

<{{examples/credential_request_jwt_vc_json_with_claims.json}}

#### Credential Response {#credential_response_jwt_vc_json}

The value of the `credential` claim in the Credential Response MUST be a JWT. Credentials of this format are already a sequence of base64url-encoded values separated by period characters and MUST NOT be re-encoded. 

The following is a non-normative example of a Credential Response with Credential format `jwt_vc_json`:

<{{examples/credential_response_jwt_vc_json.txt}}

### VC Secured using Data Integrity, using JSON-LD, with Proof Suite Requiring Linked Data Canonicalization

#### Format Identifier

The Credential format identifier is `ldp_vc`.

When the `format` value is `ldp_vc`, entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules. 

`@context` value in the `credential_definition` object could be used by the Wallet to check whether it supports a certain VC or not. If necessary, the Wallet could apply JSON-LD processing to the Credential issued by the Credential Issuer.

Note: Data Integrity used to be called Linked Data Proofs, hence "ldp" in the Credential format identifier.

#### Credential Issuer Metadata {#server_metadata_ldp_vc}

The following additional Credential Issuer metadata are defined for this Credential format to be added to the `credential_configurations_supported` parameter in addition to those defined in (#credential-issuer-parameters):

* `credential_definition`: REQUIRED. Object containing the detailed description of the Credential type. It consists at least of the following three sub claims:
  * `@context`: REQUIRED. Array as defined in [@VC_DATA], Section 4.1.
  * `type`: REQUIRED. Array designating the types a certain credential type supports according to [@VC_DATA], Section 4.3.
  * `credentialSubject`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. To express the specifics about the claim, the most deeply nested value MAY be an object that includes a following non-exhaustive list of parameters defined by this specification:
 * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the Credential Issuer will always include this claim in the issued Credential. If set to `false`, the claim is not included in the issued Credential if the wallet did not request the inclusion of the claim, and/or if the Credential Issuer chose to not include the claim. If the `mandatory` parameter is omitted, the default should be assumed to be `false`.
      * `value_type`: OPTIONAL. String value determining the type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
      * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
          * `name`: OPTIONAL. String value of a display name for the claim.
          * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. Array of the claim name values that lists them in the order they should be displayed by the Wallet.

It is recommended to define an `@context` value to communicate additional information such as which claims are mandatory-to-be-issued, type of claim value (i.e., string, number, etc.), display properties of a Credential and the order of the claim values when displayed as in (#vc-jwt).

The following is a non-normative example of an object comprising `credential_configurations_supported` parameter of Credential format `ldp_vc`:

<{{examples/credential_metadata_ldp_vc.json}}

#### Authorization Details {#authorization_ldp_vc}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.  

* `credential_definition`: OPTIONAL. Object containing the detailed description of the Credential consisting of the following sub claim:
    * `credentialSubject`: OPTIONAL. Object as defined in (#authorization_jwt_vc_json).

Note that the `@context` and `type` are referenced in the `credential_configurations_supported` object in the Credential Issuer metadata.

The following is a non-normative example of an authorization details object with Credential format `ldp_vc`:

<{{examples/authorization_details_ldp_vc.json}}

#### Credential Request {#credential_request_ldp_vc}

The following additional parameters are defined for Credential Requests and this Credential format.  

* `credential_definition`: REQUIRED when the `credential_identifier` is not present in the Credential Request. MUST NOT be used otherwise. Object containing the detailed description of the Credential type. It consists at least of the following sub claims:
  * `@context`: REQUIRED. Array as defined in (#server_metadata_ldp_vc).
  * `type`: REQUIRED. Array as defined in (#server_metadata_ldp_vc). The Credential issued by the Credential Issuer MUST at least contain the values listed in this claim.
  * `credentialSubject`: OPTIONAL. Object as defined in (#authorization_ldp_vc).

The following is a non-normative example of a Credential Request with Credential format `ldp_vc`:

<{{examples/credential_request_ldp_vc.json}}

The following is a non-normative example of a Credential request with the key proof type `ldp_vp`:

<{{examples/credential_request_ldp_vc_vp.json}}

#### Credential Response

The value of the `credential` claim in the Credential Response MUST be a JSON object. Credentials of this format MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response with Credential format `ldp_vc`:

<{{examples/credential_response_ldp_vc.txt}}

### VC signed as a JWT, Using JSON-LD

#### Format Identifier

The Credential format identifier is `jwt_vc_json-ld`.

When the `format` value is `jwt_vc_json-ld`, entire Credential Offer, Authorization Details, Credential Request and Credential Issuer metadata, including `credential_definition` object, MUST NOT be processed using JSON-LD rules.

`@context` value in the `credential_definition` object could be used by the Wallet to check whether it supports a certain VC or not. If necessary, the Wallet could apply JSON-LD processing to the Credential issued by the Credential Issuer.

#### Credential Issuer Metadata

The definitions in (#server_metadata_ldp_vc) apply for metadata of Credentials of this type as well. 

#### Authorization Details

The definitions in (#authorization_ldp_vc) apply for credentials of this type as well.

#### Credential Request

The definitions in (#credential_request_ldp_vc) apply for Credentials of this type as well.

#### Credential Response

The definitions in (#credential_response_jwt_vc_json) apply for Credentials of this type as well.

## ISO mDL

This section defines a Credential Format Profile for Credentials complying with [@!ISO.18013-5].

### Format Identifier

The Credential format identifier is `mso_mdoc`.

### Credential Issuer Metadata {#server_metadata_mso_mdoc}

The following additional Credential Issuer metadata are defined for this Credential format to be added to the `credential_configurations_supported` parameter in addition to those defined in (#credential-issuer-parameters).

* `doctype`: REQUIRED. String identifying the Credential type as defined in [@!ISO.18013-5].
* `claims`: OPTIONAL. Object containing a list of name/value pairs, where the name is a certain `namespace` as defined in [@!ISO.18013-5] (or any profile of it), and the value is an object. This object also contains a list of name/value pairs, where the name is a claim name value that is defined in the respective namespace and is offered in the Credential. The value is an object detailing the specifics of the claim with the following non-exhaustive list of parameters that MAY be included:
    * `mandatory`: OPTIONAL. Boolean which, when set to `true`, indicates that the Credential Issuer will always include this claim in the issued Credential. If set to `false`, the claim is not included in the issued Credential if the wallet did not request the inclusion of the claim, and/or if the Credential Issuer chose to not include the claim. If the `mandatory` parameter is omitted, the default should be assumed to be `false`.
    * `value_type`: OPTIONAL. String value determining the type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
    * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. Array of namespaced claim name values that lists them in the order they should be displayed by the Wallet. The values MUST be two strings separated by a tilde ('~') character, where the first string is a namespace value and a second is a claim name value. For example, `org.iso.18013.5.1~given_name".

The following is a non-normative example of an object comprising `credential_configurations_supported` parameter of Credential format `mso_mdoc`:

<{{examples/credential_metadata_mso_mdoc.json}}

### Authorization Details

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.

* `claims`: OPTIONAL. Object as defined in (#server_metadata_mso_mdoc).

Note that the `doctype` is referenced in the `credential_configurations_supported` object in the Credential Issuer metadata.

The following is a non-normative example of an authorization details object with Credential format `mso_mdoc`:

<{{examples/authorization_details_mso_doc.json}}

### Credential Request

The following additional parameters are defined for Credential Requests and this Credential format.  

* `doctype`: REQUIRED when the `credential_identifier` is not present in the Credential Request. MUST NOT be used otherwise. String as defined in (#server_metadata_mso_mdoc). The Credential issued by the Credential Issuer MUST at least contain the values listed in this claim.
* `claims`: OPTIONAL. Object as defined in (#server_metadata_mso_mdoc).

The following is a non-normative example of a Credential Request with Credential format `mso_mdoc`:

<{{examples/credential_request_iso_mdl_with_claims.json}}

### Credential Response

The value of the `credential` claim in the Credential Response MUST be a string that is the base64url-encoded representation of the issued Credential.

## IETF SD-JWT VC

This section defines a Credential Format Profile for Credentials complying with [@!I-D.ietf-oauth-sd-jwt-vc].

### Format Identifier

The Credential format identifier is `vc+sd-jwt`.

### Credential Issuer Metadata {#server_metadata_sd_jwt_vc}

The following additional Credential Issuer metadata parameters are defined for this Credential format to be added to the `credentials_supported` parameter in addition to those defined in (#credential-issuer-parameters).


* `vct`: REQUIRED. String designating the type of a Credential as defined in [@!I-D.ietf-oauth-sd-jwt-vc].
* `claims`: OPTIONAL. Object containing a list of name/value pairs, where each name identifies a claim about the subject offered in the Credential. The value can be another such object (nested data structures), or an array of such objects. To express the specifics about the claim, the most deeply nested value MAY be an object that includes a following non-exhaustive list of parameters defined by this specification:
    * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued Credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
    * `value_type`: OPTIONAL. String value determining type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
    * `display`: OPTIONAL. Array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object for each language identifier.
* `order`: OPTIONAL. An array of the claim name values that lists them in the order they should be displayed by the Wallet.

The following is a non-normative example of an object comprising `credentials_supported` parameter of Credential format `vc+sd-jwt`.

<{{examples/credential_metadata_sd_jwt_vc.json}}

### Authorization Details {#authorization_sd_jwt_vc}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.

* `vct`: REQUIRED. String as defined in (#server_metadata_sd_jwt_vc). This claim contains the type values the Wallet requests authorization for at the Credential Issuer.
* `claims`: OPTIONAL. An object as defined in (#server_metadata_sd_jwt_vc).

The following is a non-normative example of an authorization details object with Credential format `vc+sd-jwt`.

<{{examples/authorization_details_sd_jwt_vc.json}}

### Credential Request

The following additional parameters are defined for Credential Requests and this Credential format.

* `vct`: REQUIRED when the `credential_identifier` is not present in the Credential Request. MUST NOT be used otherwise. String as defined in (#server_metadata_sd_jwt_vc). This claim contains the type value of the Credential the Wallet requests the Credential Issuer to issue.
* `claims`: OPTIONAL. An object as defined in (#server_metadata_sd_jwt_vc).

The following is a non-normative example of a Credential Request with Credential format `vc+sd-jwt`.

<{{examples/credential_request_sd_jwt_vc.json}}

### Credential Response {#credential_response_jwt_vc_json}

The value of the `credential` claim in the Credential Response MUST be a string that is an SD-JWT VC. Credentials of this format are already suitable for transfer and, therefore, they need not and MUST NOT be re-encoded.

# Document History

   [[ To be removed from the final specification ]]
   
   -13

   * added a Notification Endpoint used by the Wallet to notify the Credential Issuer of certain events for issued Credentials
   * completed IANA registrations section
   * clarified description of a `mandatory` claim
   * made sure to use gender-neutral language throughout the specification
   * changed `authorization_details` to use `credential_configuration_id` pointing to the name of a `credential_configurations_supported` object in the Credential Issuer's Metadata
   * renamed `credentials` Credential Offer parameter to `credential_configurations`
   * renamed `credentials_supported` Credential Issuer metadata parameter to `credential_configurations_supported`
   * grouped `credential_encryption_jwk`, `credential_response_encryption_alg` and `credential_response_encryption_enc` from Credential Request into a single `credential_response_encryption` object
   * replaced `user_pin_required` in Credential Offer with a `tx_code` object that also now contains `description` and `length`
   * reworked flow description in Overview section
   * removed Credential Offer examples from Credential format profiles
   * added support for HTTP Accept-Language Header in the request for Credential Issuer Metadata to request a subset for display data
   * clarified how the Credential Issuer indicates that it requires proof of possession of the cryptographic key material in the Credential Request
   * added an option to use data integrity proofs as proof of possession of the cryptographic key material in the Credential Request
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
