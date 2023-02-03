%%%
title = "OpenID for Verifiable Credential Issuance"
abbrev = "openid-4-verifiable-credential-issuance"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-4-verifiable-credential-issuance-1_0-11"
status = "standard"

[[author]]
initials="T."
surname="Lodderstedt"
fullname="Torsten Lodderstedt"
organization="yes.com"
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

This specification defines an API that is used to issue verifiable credentials. W3C formats [@VC_DATA] as well as other Credential formats, like [@ISO.18013-5], are supported. 

Verifiable Credentials are very similar to identity assertions, like ID Tokens in OpenID Connect [@OpenID.Core], in that they allow a Credential Issuer to assert End-User claims. However, in contrast to the identity assertions, a verifiable credential follows a pre-defined schema (the Credential type) and is bound to a certain holder, e.g trough cryptographic holder binding. This allows secure direct presentation of the Credential from the End-User to the RP, without involvement of the Credential Issuer. This specification caters for those differences.

Access to this API is authorized using OAuth 2.0 [@!RFC6749], i.e. The Wallet uses OAuth 2.0 to obtain authorization to receive verifiable credentials. This way the issuance process can benefit from the proven security, simplicity, and flexibility of OAuth 2.0 and existing OAuth 2.0 deployments and OpenID Connect OPs (see [@OpenID.Core]) can be extended to become Credential Issuers. 

# Terminology

This specification uses the terms "Access Token", "Authorization Endpoint", "Authorization Request", "Authorization Response", "Authorization Code Grant", "Authorization Server", "Client", "Client Authentication", "Client Identifier", "Grant Type", "Refresh Token", "Token Endpoint", Token Request" and "Token Response" defined by OAuth 2.0 [@!RFC6749], the terms "End-User", "Entity", and "Request Object" as defined by OpenID Connect Core [@!OpenID.Core], the term "JSON Web Token (JWT)" defined by JSON Web Token (JWT) [@!RFC7519], the term "JOSE Header" and "Base64url Encoding" defined by JSON Web Signature (JWS) [@!RFC7515].

This specification also defines the following terms. In the case where a term has a definition that differs, the definition below is authoritative.

Credential:
:  A set of one or more claims about a subject made by a Credential Issuer. Note that this definition of a term "credential" in this specification is different from that in [@!OpenID.Core] and [@!RFC6749].

Verifiable Credential (VC):
:  An Issuer-signed Credential whose authenticity can be cryptographically verified. Can be of any format used in the Issuer-Holder-Verifier Model, including, but not limited to those defined in [@VC_DATA] and [@ISO.18013-5].

W3C Verifiable Credential:
:  A Verifiable Credential compliant to the [@VC_DATA] specification.

Presentation:
: Data that is shared with a specific verifier, derived from one or more Verifiable Credentials that can be from the same or different issuers.

Verifiable Presentation (VP):
:  A Holder-signed Credential whose authenticity can be cryptographically verified to provide Cryptographic Holder Binding. Can be of any format used in the Issuer-Holder-Verifier Model, including, but not limited to those defined in [@VC_DATA] and [@ISO.18013-5].

W3C Verifiable Presentation:
:  A Verifiable Presentations compliant to the [@VC_DATA] specification.

Credential Issuer:
:  An entity that issues Verifiable Credentials. Also called Issuer. In the context of this specification, the Credential Issuer acts as an OAuth 2.0 Authorization Server (see [@!RFC6749]).

Holder:
:  An entity that receives Verifiable Credentials and has control over them to present them to the Verifiers as Verifiable Presentations.

Verifier:
:  An entity that requests, receives and validates Verifiable Presentations. During presentation of Credentials, Verifier acts as an OAuth 2.0 Client towards the Wallet that is acting as an OAuth 2.0 Authorization Server. The Verifier is a specific case of OAuth 2.0 Client, just like Relying Party (RP) in [@OpenID.Core].

Issuer-Holder-Verifier Model:
:  A model for claims sharing where claims are issued in the form of Verifiable Credentials independent of the process of presenting them as Verifiable Presentation to the Verifiers. An issued Verifiable Credential can (but must not necessarily) be used multiple times.

Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential.

Cryptographic Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proving control over the same private key during the issuance and presentation. Mechanism might depend on the Credential Format. For example, in `jwt_vc_json` Credential Format, a VC with Cryptographic Holder Binding contains a public key or a reference to a public key that matches to the private key controlled by the Holder.

Claim-based Holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by proofing certain claims, e.g. name and date of birth, for example by presenting another Verifiable Credential. Claim-based Holder Binding allows long term, cross-device use of a credential as it does not depend on cryptographic key material stored on a certain device. One example of such a Verifiable Credential could be a Diploma.

Biometrics-based holder Binding:
:  Ability of the Holder to prove legitimate possession of a Verifiable Credential by demonstrating a certain biometric trait, such as finger print or face. One example of a Verifiable Credential with biometric holder binding is a mobile drivers license [@ISO.18013-5], which contains a portrait of the holder.

Wallet:
:  An entity used by the Holder to receive, store, present, and manage Verifiable Credentials and key material. There is no single deployment model of a Wallet: Verifiable Credentials and keys can both be stored/managed locally, or by using a remote self-hosted service, or a remote third-party service. In the context of this specification, the Wallet acts as an OAuth 2.0 Authorization Server (see [@!RFC6749]) towards the Credential Verifier which acts as the OAuth 2.0 Client.

Deferred Credential Issuance:
:  Issuance of Credentials not directly in the response to a Credential issuance request, but following a period of time that can be used to perform certain offline business processes.

# Overview

## Credential Issuer

This specification defines an API for credential issuance provided by a Credential Issuer. The API is comprised of the following endpoints:

* A mandatory Credential Endpoint from which Credentials can be issued.  See (#credential-endpoint).
* An optional Batch Credential Endpoint from which multiple Credentials can be issued in one request. See (#batch-credential-endpoint).
* An optional Deferred Credential Endpoint to allow for the deferred delivery of credentials (#deferred-credential-issuance). 
* An optional mechanism for the Credential Issuer to make a Credential Offer to the Wallet to encourage the Wallet to start the issuance flow. See (#issuance_initiation_endpoint).
* A mechanism for the Credential Issuer to publish metadata about the Credentials it is capable of issuing. See (#credential-issuer-metadata)

Both the Credential and the Batch Credential Endpoints have the (optional) ability to bind an issued Credential to certain cryptographic key material. Both requests therefore allow to convey a proof of posession for the key material. Multiple proof types are supported. 

## OAuth 2.0

Every Credential Issuer utilizes an OAuth 2.0 [@!RFC6749] Authorization Server to authorize access. The same OAuth 2.0 Authorization Server can protect one or more Credential Issuers. Wallets determine the Authorization Server a certain Credential Issuer relies on using the Credential Issuer's metadata (see (#credential-issuer-metadata)).   

All OAuth 2.0 Grant Types and extensions mechanisms can be used in conjunction with the credential issuance API. Aspects not defined in this specification are expected to follow [@!RFC6749]. 

Existing OAuth 2.0 mechanisms are extended as following:

* A new Grant Type "Pre-Authorized Code" along with additional token response paramters `authorization_pending` and `interval` is defined to facilitate flows where the preparation of the credential issuance is conducted before the actual OAuth flow starts (#pre-authz-code-flow).
* A new authorization details [@!I-D.ietf-oauth-rar] type `openid_credential` is defined to convey the details about the Credentials (including formats and types) the Wallet wants to obtain (#authorization-details). 
* Client metadata is used to convey Wallet's metadata. A new metadata parameter `credential_offer_endpoint` is added to allow a Wallet (acting as OAuth 2.0 client) to publish its Credential Offer Endpoint (#client-metadata).
* Authorization Endpoint: An additional parameter `issuer_state` is added to convey state in the context of processing an issuer-initiated credential offer (#credential-authz-request). Additional parameters `wallet_issuer` and `user_hint` are added to enable the Credential Issuer to request Verifiable Presentations from the calling Wallet in the course of Authorization Request processing. 
* Token Endpoint: optional response parameters `c_nonce` and `c_nonce_expires_in` are added to the Token Endpoint to provide the client with a nonce to be used for proof of possession of key material in a subsequent request to the Credential Endpoint (#token-response). 

## Core Concepts

The Wallet sends one Credential Request per individual Credential to the Credential Endpoint. The Wallet MAY use the same Access Token to send multiple Credential Requests to request issuance of the following:

* multiple Credentials of different types bound to the same proof, or
* multiple Credentials of the same type bound to different proofs.

The Wallet MAY send one Batch Credential Request to the Batch Credential Endpoint to request the following:

* multiple Credentials of different types bound to the same proof, or 
* multiple Credentials of the same type bound to different proofs in the Batch Credential Response.

In the course of the authorization process, the Credential Issuer MAY also request Credential presentation as means to authenticate or identify the End-User during the issuance flow as illustrated in a use case in (#use-case-2).

At its core, this specification is Credential format agnostic and allows implementers to leverage specific capabilities of Credential formats of their choice. Multiple Credential formats can be used within the same transaction. 

The specification achieves this by defining the following:

* Extension points to add Credential format specific parameters or claims in the Credential Issuer metadata, Credential Offer, Authorization Request, Credential Request and Batch Credential Request,
* Credential format identifiers to identify Credential format specific set of parameters and claims to be applied at each extention point. This set of Credential format specific set of parameters and claims is referred to as a "Credential Format Profile" in this specification.

This specification defines Credential Format Profiles for W3C Verifiable Credentials as defined in [@VC_DATA] and ISO/IEC 18013-5 mDL as defined in [@ISO.18013-5] in (#format_profiles) that contain Credential Format specific parameters to be included at each extension point defined in this specification. Other specifications or deployments can define their own Credential Format Profiles using above-mentioned extension points.

The issuance can have multiple characteristics, which can be combined depending on the use-cases: 

* Authorization Code Flow or Pre-Authorized Code Flow: the Credential Issuer can obtain user information to turn into a verifiable Credential using user authentication and consent at the Credential Issuer's Authorization Endpoint (Authorization Code Flow), or using out of bound mechanisms outside of the issuance flow (Pre-Authorized Code Flow)
* Wallet initiated or Issuer initiated: the request from the Wallet can be sent to the Credential Issuer without any gesture from the Credential Issuer (Wallet Initiated), or following the communication from the Credential Issuer (Issuer Initiated).
* Same-device or Cross-device: the Credential Issuer's user experience (website or an app) can reside on the same device, or on different devices with the Wallet to which the Credential is issued.
* Just-in-time or Deferred: the Credential Issuer can issue the Credential directly in response to the Credential Request (just-in-time), or requires time and needs the Wallet to come back to retrieve Credential (deferred).

The following sub-sections illusterate some of the authorization flows supported by this specification.

## Authorization Code Flow {#authorization-code-flow}

Below is a diagram of a Credential issuance using the Authorization Code flow, using grant type `authorization_code` as defined in [@!RFC6749].

The diagram shows how a Wallet initiated flow as described in use case (#use-case-1) is implemented with the Credential Issuance API defined in this specification. Note that the diagram and does not illustrate all of the optional features of this specification. 

!---
~~~ ascii-art
+--------------+   +-----------+                                         +-------------+
| User         |   |   Wallet  |                                         |   Issuer    |
+--------------+   +-----------+                                         +-------------+  
        |    interacts   |                                                      |
        |--------------->|                                                      |
        |                |  (1) Obtains Issuer's Credential Issuer metadata     |
        |                |<---------------------------------------------------->|
        |                |                                                      |
        |                |  (2) Authorization Request                           |
        |                |      (type(s) of Credentials to be issued)           |
        |                |----------------------------------------------------->|
        |                |                                                      |
        |   User Authentication / Consent                                       |
        |                |                                                      |
        |                |  (3)   Authorization Response (code)                 |
        |                |<-----------------------------------------------------|
        |                |                                                      |
        |                |  (4) Token Request (code)                            |
        |                |----------------------------------------------------->| 
        |                |      Token Response (access_token)                   |
        |                |<-----------------------------------------------------|    
        |                |                                                      |
        |                |  (5) Credential Request (access_token, proof(s))     |
        |                |----------------------------------------------------->| 
        |                |      Credential Response                             |
        |                |      (credential(s) OR acceptance_token)             |
        |                |<-----------------------------------------------------|             
~~~
!---
Figure: Issuance using Authorization code flow 

(1) The wallet uses the Credential Issuer's metadata (#credential-issuer-metadata) to learn what credential types and formats the Credential Issuer supports and to determine the issuer URL of the OAuth authorization server the Credential Issuer relies on. Note in this example, the Credential Issuer also provides the OAuth Authorization Server. This specification enables deployments where the Credential Issuer API and the Authorization Server are different services, perhaps even provided by different entities.  

(2) The Wallet sends an Authorization Request to the Authorization Endpoint. The Authorization Endpoint processes the Authorization Request, which typically includes user authentication and gathering of user consent. 

(3) The Authorization Endpoint returns an Authorization Response with the Authorization Code upon successfully processing the Authorization Request. 

Note that steps (2) and (3) happens in the frontchannel, by redirecting the End-User via the User Agent. Those steps are defined in (#authorization_endpoint).

(4) The Wallet sends a Token Request to the Token Endpoint with the Authorization Code obtained in step (3). The Token Endpoint returns an Access Token in the Token Response upon successfully validating Authorization Code. This step happens in the backchannel using server to server communication. This step is defined in (#token_endpoint).

(5) The Wallet sends a Credential Request to the Credential Issuer's Credential Endpoint with the Access Token and (optionally) the proof of possession of the public key to which the the issued VC shall be bound to. Upon successfully validating Access Token and proof, the Credential Issuer returns a VC in the Credential Response if it is able to issue a Credential right away. This step is defined in (#credential-endpoint).

If the Credential Issuer requires more time to issue a Credential, the Credential Issuer MAY return an Acceptance Token to the Wallet with the information when the Wallet can start sending Deferred Credential Request to obtain an issued Credential as defined in (#deferred-credential-issuance).

If the Issuer wants to issue multiple Credentials in one response, the Issuer MAY support the Batch Credential Endpoint and the Wallet MAY send a Batch Credential Request to the Batch Credential Endpoint as defined in (#batch-credential-endpoint).

With grant type `authorization_code`, it is RECOMMENDED to use PKCE as defined in [@!RFC7636] to prevent authorization code interception attacks and Pushed Authorization Requests [@RFC9126] to ensure integrity and authenticity of the authorization request.

Note: this flow is based on OAuth 2.0 and the Authorization Code Grant type, but this specification can be used with other OAuth grant types as well. 

## Pre-Authorized Code Flow {#pre-authz-code-flow}

Figure 2 is a diagram of a Credential issuance using the Pre-Authorized Code Flow. In this flow, before initiating the flow with the Wallet, the Credential Issuer first conducts the steps required to prepare the Credential issuance, e.g. user authentication and authorization. Consequently, the Pre-Authorized Code is sent by the Credential Issuer to the Wallet. This flow does not use the Authorization Endpoint, and The Wallet exchanges the Pre-Authorized Code for the Access Token directly at the Token Endpoint. Access Toke is than used to request Credential issuance at the Credential Endpoint. See (#use-case-4) for a use case.

How the End-User provides information required for the issuance of a requested Credential to the Credential Issuer and the business processes conducted by the Credential Issuer to prepare a Credential are out of scope of this specification.

This flow uses the newly defined OAuth 2.0 Grant Type "urn:ietf:params:oauth:grant-type:pre-authorized_code".

The diagram is based on a Credential Issuer initiated flow illustrated in a use case in (#use-case-4) and does not illustrate all of the optional features.

!---
~~~ ascii-art
+--------------+   +-----------+                                         +-------------+
| User         |   |   Wallet  |                                         |   Issuer    |
+--------------+   +-----------+                                         +-------------+
        |                |  (1) User provides  information required             |  
        |                |      for the issuance of a certain Credential        |
        |---------------------------------------------------------------------->|
        |                |                                                      |
        |                |  (2) Credential Offer (Pre-Authorized Code)          |
        |                |<-----------------------------------------------------|        
        |                |  (3) Obtains Issuer's Credential Issuer metadata     |
        |                |<---------------------------------------------------->|
        |   interacts    |                                                      |
        |--------------->|                                                      |
        |                |                                                      |
        |                |  (4) Token Request (Pre-Authorized Code, pin)        |
        |                |----------------------------------------------------->| 
        |                |      Token Response (access_token)                   |
        |                |<-----------------------------------------------------|    
        |                |                                                      |
        |                |  (5) Credential Request (access_token, proof(s))     |
        |                |----------------------------------------------------->| 
        |                |      Credential Response                             |
        |                |      (credential(s))                                 |
        |                |<-----------------------------------------------------|             
~~~
!---
Figure: Issuance using Pre-Authorized code flow 

(1) the Credential Issuer successfully obtains consent and user data required for the issuance of a requested Credential from the End-User using Issuer specific business process.

(2) The flow defined in this specification begins as the Credential Issuer generates a Credential Offer for certain Credential(s) and communicates it to the Wallet, for example as a QR code or as a deeplink. 

(3) The Wallet uses information from the Credential Offer to obtain the Credential Issuer's metadata including details about the Credential that this Credential Issuer wants to issue. This step is defined in (#issuance_initiation_endpoint).

(4) The Wallet sends the Pre-Authorized Code obtained in step (1) to the Token Request. If the Credential Issuer required so, the Wallet sends a PIN, it has previously obtained from the user with the request, This step is defined in (#token_endpoint).  

(5) This step is the same as Step 5 in the Authorization Code Flow. 

It is important to note that anyone who possesses a valid Pre-Authorized Code, without further security measures, would be able to receive a VC from the Credential Issuer. Implementers MUST implement mitigations most suitable to the use-case. 

One such mechanism defined in this specification is the usage of PIN. If in the Credential Offer the Credential Issuer indicated that the PIN is required, the End-User is requested to type in a PIN sent via a channel different that the issuance Flow and the PIN is sent to the Credential Issuer in the Token Request. 

For more details and concrete mitigations, see (#security_considerations_pre-authz-code).

# Credential Offer Endpoint {#issuance_initiation_endpoint}

This endpoint is used by a Credential Issuer in case it is already in an interaction with a user that wishes to initate a Credential issuance. It is used to pass available information relevant for the Credential issuance to ensure a convenient and secure process. 

## Credential Offer {#issuance_initiation_request}

The Credential Issuer sends Credential Offer as an HTTP GET request or an HTTP redirect to the Credential Offer Endpoint URL defined in (#client-metadata).

Credential Offer object which is a JSON object with the Credential Offer parameters, can be sent by value or by refernce.

Credential Offer contains a single URI query parameter `credential_offer` or `credential_offer_uri`:

* `credential_offer`: CONDITIONAL. A JSON object with the Credential Offer parameters. MUST NOT be present when `credential_offer_uri` parameter is present.
* `credential_offer_uri`: A URL using the `https` scheme referencing a resource containing a JSON object with the Credential Offer parameters. MUST NOT be present when `credential_offer` parameter is present.

The Credential Issuer MAY render a QR code containing the Credential Offer that can be scanned by the End-User using a Wallet, or a deeplink that the End-User can click.

For security considerations, see (#credential-offer-security).

### Credential Offer Parameters {#credential_offer_parameters}

Credential Offer object MAY contain the following parameters: 

* `credential_issuer`: REQUIRED. The URL of the Credential Issuer, the Wallet is requested to obtain one or more Credentials from. 
* `credentials`: REQUIRED. A JSON array, where every entry is a JSON object or a JSON string. If the entry is an object, the object contains the data related to a certain credential type the Wallet MAY request. Each object MUST contain a `format` Claim determining the format of the credential to be requested and further parameters characterising the type of the credential to be requested as defined in (#format_profiles). If the entry is a string, the string value MUST be one of the `id` values in one of the objects in the `credentials_supported` Credential Issuer metadata parameter. When processing, the Wallet MUST resolve this string value to the respective object.
* `grants`: OPTIONAL. A JSON object indicating to the Wallet the Grant Types the Credential Issuer's AS is prepared to process for this credential offer. Every grant is represented by a key and an object. The key value is the Grant Type identifier, the object MAY contain parameters either determining the way the Wallet MUST use the particular grant and/or parameters the Wallet MUST send with the respective request(s). If `grants` is not present or empty, the Wallet MUST determine the Grant Types the Credential Issuer's AS supports using the respective metadata. When multiple grants are present, it's at the Wallet’s discretion which one to use.

The following values are defined by this specification: 

* Grant Type `authorization_code`:
  * `issuer_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the Wallet that is used to bind the subsequent Authorization Request with the Credential Issuer to a context set up during previous steps. If the Wallet decides to use the Authorization Code Flow and received a value for this parameter, it MUST include it in the subsequent Authorization Request to the Credential Issuer as the `issuer_state` parameter value. 
* Grant Type `urn:ietf:params:oauth:grant-type:pre-authorized_code`:
  * `pre-authorized_code`: REQUIRED. The code representing the Credential Issuer's authorization for the Wallet to obtain Credentials of a certain type. This code MUST be short lived and single-use. If the Wallet decides to use the Pre-Authorized Code Flow, this parameter value MUST be include in the subsequent Token Request with the Pre-Authorized Code Flow.
  * `user_pin_required`: OPTIONAL. Boolean value specifying whether the Credential Issuer expects presentation of a user PIN along with the Token Request in a Pre-Authorized Code Flow. Default is `false`. This PIN is intended to bind the Pre-Authorized Code to a certain transaction in order to prevent replay of this code by an attacker that, for example, scanned the QR code while standing behind the legit user. It is RECOMMENDED to send a PIN via a separate channel. If the Wallet decides to use the Pre-Authorized Code Flow, a PIN value MUST be sent in the `user_pin` parameter with the respective Token Request. 

The following non-normative example shows a Credential Offer object where the Credential Issuer can offer the issuance of two Credentials of different formats, one as JSON string ("UniversityDegree_JWT") and the other one as JSON object:

<{{examples/credential_offer_multiple_credentials.json}}

Note that the examples throughout the specification use Credential Format specific parameters defined in the Credential Format Profiles that can be found in (#format_profiles).

### Sending Credential Offer by Value Using `credential_offer` Parameter

Below is a non-normative example of a Credential Offer passed by value:

```
  GET /credential_offer?credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.example.com
  %22,%22credentials%22:%5B%7B%22format%22:%22jwt_vc_json%22,%22types%22:%5B%22Verifiabl
  eCredential%22,%22UniversityDegreeCredential%22%5D%7D%5D,%22issuer_state%22:%22eyJhbGciOiJS
  U0Et...FYUaBy%22%7D
```

The following is a non-normative example of a Credential Offer that can be included in a QR code or a deeplink used to invoke Wallet deployed as a native app:

```
openid-credential-offer://credential_offer=%7B%22credential_issuer%22:%22https://credential-issuer.example.com
%22,%22credentials%22:%5B%7B%22format%22:%22jwt_vc_json%22,%22types%22:%5B%22VerifiableCr
edential%22,%22UniversityDegreeCredential%22%5D%7D%5D,%22issuer_state%22:%22eyJhbGciOiJSU0Et...
FYUaBy%22%7D
```

### Sending Credential Offer by Reference Using `credential_offer_uri` Parameter

Upon receipt of the `credential_offer_uri`, the Wallet MUST send an HTTP GET request to URI to retrieve the referenced Credential Offer Object, unless it is already cached, and parse it to recreate the Credential Offer parameters.

Note: The Credential Issuer SHOULD use a unique URI for each Credential Offer utilizing distinct parameters, or otherwise prevent the Credential Issuer from caching the `credential_offer_uri`.

Below is a non-normative example of this fetch process:

```
GET /credential-offer.jwt HTTP/1.1
Host: server.example.com
```

Response from the Credential Issuer that contains a Credential Offer Object MUST have the media type "application/json".

This ability to pass Credential Offer by reference is particularly useful for large requests.

Below is a non-normative example of the Credential Offer displayed by the Credential Issuer as a QR code when the Credential Offer is passed by reference:

```
openid-credential-offer://?
  credential_offer_uri=https%3A%2F%2Fserver%2Eexample%2Ecom%2Fcredential-offer.jwt
```

Below is a non-normative example of a response from the Credential Issuer that contains a Credential Offer Object used to encourage the Wallet to start an Authorization Code Flow:

<{{examples/credential_offer_authz_code.txt}}

Below is a non-normative example how a Credential Offer Object might look like for a Pre-Authorized Code Flow (with a credential type reference):

<{{examples/credential_offer_by_reference.json}}

## Credential Offer Response

The Wallet is not supposed to create a response. UX control stays with the Wallet after completion of the process. 

# Authorization Endpoint {#authorization_endpoint}

The Authorization Endpoint is used in the same manner as defined in [@!RFC6749] taking into account the recommendations given in [@!I-D.ietf-oauth-security-topics].

## Authorization Request {#credential-authz-request}

An Authorization Request is an OAuth 2.0 Authorization Request as defined in section 4.1.1 of [@!RFC6749], which requests to grant access to the Credential Endpoint as defined in (#credential-endpoint). 

There are two possible ways to request issuance of a specific Credential type in an Authorization Request. One way is to use of the `authorization_details` request parameter as defined in [@!I-D.ietf-oauth-rar] with one or more authorization details objects of type `openid_credential` (#authorization-details). The other is through the use of scopes as defined in (#credential-request-using-type-specific-scope).

### Request Issuance of a Certain Credential Type using `authorization_details` Parameter {#authorization-details}

The request parameter `authorization_details` defined in Section 2 of [@!I-D.ietf-oauth-rar] MUST be used to convey the details about the Credentials the Wallet wants to obtain. This specification introduces a new authorization details type `openid_credential` and defines the following elements to be used with this authorization details type:

* `type` REQUIRED. JSON string that determines the authorization details type. MUST be set to `openid_credential` for the purpose of this specification.
* `format`: REQUIRED. JSON string representing the format in which the Credential is requested to be issued. This Credential format identifier determines further claims in the authorization details object specifically used to identify the Credential type to be issued. This specification defines Credential Format Profiles in (#format_profiles).  

A non-normative example of an `authorization_details` object. 

<{{examples/authorization_details.json}}

If the Credential Issuer metadata contains an `authorization_server` parameter, the authorization detail's `locations` common data field MUST be set to the Credential Issuer Identifier value. A non-normative example for a deployment where an AS protects multiple Credential Issuers would look like this:

<{{examples/authorization_details_with_as.json}}

Below is a non-normative example of an Authorization Request using the `authorization_details` parameter that would be sent by the User Agent to the Authorization Server in response to an HTTP 302 redirect response by the Wallet (with line wraps within values for display purposes only):

```
GET /authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &authorization_details=%5B%7B%22type%22:%22openid_credential
  %22,%22format%22:%22jwt_vc_json%22,%22types%22:%5B%22Verifia
  bleCredential%22,%22UniversityDegreeCredential%22%5D%7D%5D
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
Host: https://server.example.com
```

This non-normative example requests authorization to issue two different Credentials:

<{{examples/authorization_details_multiple_credentials.json}}

Note: applications MAY combine authorizarion details of type `openid_credential` with any other authorization details type in an Authorization Request.

### Using `scope` Parameter to Request Issuance of a Credential {#credential-request-using-type-specific-scope}

In addition to a mechanism defined in (#credential-authz-request), Credential Issuers MAY support requesting authorization to issue a credential using OAuth 2.0 scope values.

The concrete scope values and the mapping between a certain scope value and the credential type (and further parameters associated with the authorization to issue this credential type) is out of scope of this specification. Possible options include normative text in a profile of this specification defining scope values along with a description of their semantics or machine readable definitions in the Credential Issuer's metadata such as mapping a scope value to an equivalent authorization details object (see above). 

It is RECOMMENDED to use collision-resistant scope values.

Credential Issuers MUST interpret each scope value as a request to access the Credential Endpoint as defined in (#credential-endpoint) for the issuance of a Credential type identified by that scope value. Multiple scope values MAY be present in a single request whereby each 
occurrence MUST be interpreted individually.

Credential Issuers MUST ignore unknown scope values in a request.

If the Credential Issuer metadata contains an `authorization_server` property, it is RECOMMENDED to use a `resource` parameter [@!RFC8707] whose value is the Credential Issuer's identifier value to allow the AS to differentiate Credential Issuers.  

Below is a non-normative example of an Authorization Request using the scope `com.example.healthCardCredential` that would be sent by the User Agent to the Authorization Server in response to an HTTP 302 redirect response by the Wallet (with line wraps within values for display purposes only):

```
GET /authorize?
  response_type=code
  &scope=com.example.healthCardCredential
  &resource=https://credential-issuer.example.com
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
Host: https://server.example.com
```

If a scope value related to credential issuance and the `authorization_details` request parameter containing objects of type `openid_credential` are both present in a single request, the Credential Issuer MUST interpret these individually. However, if both request the same Credential type, then the Credential Issuer MUST follow the request as given by the authorization details object.

### Additional Request Parameters

This specification defines the following request parameters that can be supplied in an Authorization Request:

* `wallet_issuer`: OPTIONAL. JSON string containing the Wallet's OpenID Connect issuer URL. The Credential Issuer will use the discovery process as defined in [@!SIOPv2] to determine the Wallet's capabilities and endpoints. RECOMMENDED in Dynamic Credential Request.
* `user_hint`: OPTIONAL. JSON string containing an opaque user hint the Wallet MAY use in subsequent callbacks to optimize the user's experience. RECOMMENDED in Dynamic Credential Request.
* `issuer_state`: OPTIONAL. String value identifying a certain processing context at the Credential Issuer. A value for this parameter is typically passed in an issuance initation request from the Credential Issuer to the Wallet (see ((#issuance_initiation_request)). This request parameter is used to pass the `issuer_state` value back to the Credential Issuer. 

Note: When processing the Authorization Request, the Credential Issuer MUST take into account that the `issuer_state` is not guaranteed to originate from this Credential Issuer in all circumstances. It could have been injected by an attacker. 

### Pushed Authorization Request

Use of Pushed Authorization Requests is RECOMMENDED to ensure confidentiality, integrity, and authenticity of the request data and to avoid issues due to large requests sizes.

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

This step is OPTIONAL. After receiving an Authorization Request from the Client, the Credential Issuer MAY use this step to obtain additional Credentials from the End-User required to proceed with the authorization of the credential issuance. Credential Issuer MAY obtain an identity credential and utilize it to identify the End-User before issuing an additional credential. For a use case, see (#use-case-2).

It is RECOMMENDED that the Credential Issuer use [@OpenID4VP] to dynamically request presentation of additional Credentials. From a protocol perspective, the Credential Issuer then acts as a verifier and sends a presentation request to the Wallet. The Client SHOULD have these Credentials obtained prior to starting a transaction with this Credential Issuer. 

To enable dynamic callbacks of the Credential Issuer to the End-User's Wallet, the Wallet MAY provide additional parameters `wallet_issuer` and `user_hint` defined in the Authorization Request section of this specification.

For non-normative examples of request and response, see section 11.6 in [@OpenID4VP].

Note to the editors: need to sort out Credential Issuer's client_id with Wallet and potentially add example with `wallet_issuer` and `user_hint` 

## Successful Authorization Response

Authorization Responses MUST be made as defined in [@!RFC6749].

Below is a non-normative example of a successful Authorization Response:

```
HTTP/1.1 302 Found
  Location: https://Wallet.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
```

## Authorization Error Response

Authorization Error Response MUST be made as defined in [@!RFC6749].

When the requested scope value is invalid, unknown, or malformed, the AS should respond with the error code `invalid_scope` defined in Section 4.1.2.1 of [@!RFC6749].

Below is a non-normative example of an unsuccessful Authorization Response.

```json=
HTTP/1.1 302 Found
Location: https://client.example.net/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value
```

# Token Endpoint {#token_endpoint}

The Token Endpoint issues an Access Token and, optionally, a Refresh Token in exchange for the Authorization Code that client obtained in a successful Authorization Response. It is used in the same manner as defined in [@!RFC6749] and follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

## Token Request {#token_request}

Upon receiving a successful Authorization Response, a Token Request is made as defined in Section 4.1.3 of [@!RFC6749].

The following are the extension parameters to the Token Request used in a Pre-Authorized Code Flow:

* `pre-authorized_code`: CONDITIONAL. The code representing the authorization to obtain Credentials of a certain type. This parameter is required if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.
* `user_pin`: OPTIONAL. String value containing a user PIN. This value MUST be present if `user_pin_required` was set to `true` in the Credential Offer. The string value MUST consist of maximum 8 numeric characters (the numbers 0 - 9). This parameter MUST only be used, if the `grant_type` is `urn:ietf:params:oauth:grant-type:pre-authorized_code`.

Requirements around how the Verifier identifies and, if applicable, authenticates itself with the Wallet in the Token Request depends on the Grant Type.

For the Authorization Code Grant Type, the requirement as described in Sections 4.1.3 and 3.2.1 of [@!RFC6749] MUST be followed.

For the Pre-Authorized Code Grant Type, authentication of the client is OPTIONAL, as described in Section 3.2.1 of OAuth 2.0 [@!RFC6749] and consequently, the "client_id" is only needed when a form of Client Authentication that relies on the parameter is used.

If the Token Request contains an `authorization_details` parameter of type `openid_credential` and the Credential Issuer's metadata contains an `authorization_server` parameter, the `authorization_details` object MUST contain the Credential Issuer's identifier in the `locations` element. 

If the Token Request contains a scope value related to credential issuance and the Credential Issuer's metadata contains an `authorization_server` parameter, it is RECOMMENDED to use a `resource` parameter [@!RFC8707] whose value is the Credential Issuer's identifier value to allow the AS to differentiate Credential Issuers. 

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
  &user_pin=493536
```

## Successful Token Response {#token-response}

Token Responses are made as defined in [@!RFC6749].

In addition to the response parameters defined in [@!RFC6749], the AS MAY return the following parameters:

* `c_nonce`: OPTIONAL. JSON string containing a nonce to be used to create a proof of possession of key material when requesting a Credential (see (#credential_request)). When received, the Wallet MUST use this nonce value for its subsequent credential requests until the Credential Issuer provides a fresh nonce.
* `c_nonce_expires_in`: OPTIONAL. JSON integer denoting the lifetime in seconds of the `c_nonce`.
* `authorization_pending`: OPTIONAL. JSON Boolean. In the Pre-Authorized Code Flow, the Token Request is still pending as the Credential Issuer is waiting for the End-User interaction to complete. The client SHOULD repeat the Token Request. Before each new request, the client MUST wait at least the number of seconds specified by the `interval` response parameter. ToDo: clarify boolean.
* `interval`: OPTIONAL. The minimum amount of time in seconds that the client SHOULD wait between polling requests to the Token Endpoint in the Pre-Authorized Code Flow.  If no value is provided, clients MUST use 5 as the default.

Upon receiving Pre-Authorized Code, the Credential Issuer MAY decide to interact with the End-User in the course of the Token Request processing, which might take some time. In such a case, the Credential Issuer SHOULD respond with the error `authorization_pending` and the new return parameter `interval`.

Below is a non-normative example of a Token Response:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

  {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "token_type": "bearer",
    "expires_in": 86400,
    "c_nonce": "tZignsnFbp",
    "c_nonce_expires_in": 86400
  }
```

## Token Error Response

If the Token Request is invalid or unauthorized, the Authorization Server constructs the error response as defined as in Section 5.2 of OAuth 2.0 [@!RFC6749].

The following additional clarifications are provided for some of the error codes already defined in [@!RFC6749]:

`invalid_request`:

- the Authorization Server does not expect a PIN in the pre-authorized flow but the client provides a PIN
- the Authorization Server expects a PIN in the pre-authorized flow but the client does not provide a PIN

`invalid_grant`:

- the Authorization Server expects a PIN in the pre-authorized flow but the client provides the wrong PIN
- the End-User provides the wrong Pre-Authorized Code or the Pre-Authorized Code has expired

`invalid_client`:

- the client tried to send a Token Request with a Pre-Authorized Code without Client ID but the Authorization Server does not support anonymous access

Below is a non-normative example Token Error Response:

```json=
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
   "error": "invalid_request"
}
```

# Credential Endpoint {#credential-endpoint}

The Credential Endpoint issues a Credential as approved by the End-User upon presentation of a valid Access Token representing this approval. 

Communication with the Credential Endpoint MUST utilize TLS. 

The client can request issuance of a Credential of a certain type multiple times, e.g., to associate the Credential with different public keys/Decentralized Identifiers (DIDs) or to refresh a certain Credential.

If the Access Token is valid for requesting issuance of multiple Credentials, it is at the client's discretion to decide the order in which to request issuance of multiple Credentials requested in the Authorization Request.

## Binding the Issued Credential to the identifier of the End-User possessing that Credential {#credential-binding}

Issued Credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the Credential. Cryptographic binding allows the Verifier to verify during the presentation of a Credential that the End-User presenting a Credential is the same End-User to whom that Credential was issued. For non-cryptographic type of binding and Credentials issued without any binding, see Implementation Considerations sections (#claim-based-binding) and (#no-binding). 

Note that claims in the Credential are usually about the End-User who possesses it, but can be about another entity.

For cryptographic binding, the Client has the following options to provide cryptographic binding material for a requested Credential as defined in (#credential_request):

1. Provide proof of control alongside key material (`proof` that includes `sub_jwk` or `did`)
1. Provide only proof of control without the key material (`proof` that does not include `sub_jwk` or `did`)

## Credential Request {#credential_request}

A Client makes a Credential Request to the Credential Endpoint by sending the following parameters in the entity-body of an HTTP POST request using the "application/json" media type.

* `format`: REQUIRED. Format of the Credential to be issued. This Credential format identifier determines further parameters required to determine the type and (optionally) the content of the credential to be issued. Credential Format Profiles consisting of the Credential format specific set of parameters are defined in (#format_profiles).
* `proof`: OPTIONAL. JSON object containing proof of possession of the key material the issued Credential shall be bound to. The specification envisions use of different types of proofs for different cryptographic schemes. The `proof` object MUST contain a `proof_type` claim of type JSON string denoting the concrete proof type. This type determines the further claims in the proof object and its respective processing rules. Proof types are defined in (#proof_types). 

The `proof` element MUST incorporate a `c_nonce` value generated by the Credential Issuer and the Credential Issuer Identifier (audience) to allow the Credential Issuer to detect replay. The way that data is incorporated depends on the proof type. In a JWT, for example, the `c_nonce` is conveyd in the `nonce` claims whereas the audience is conveyed in the `aud` claim. In a Linked Data proof, for example, the `c_nonce` is included as the `challenge` element in the proof object and the Credential Issuer (the intended audience) is included as the `domain` element.

Below is a non-normative example of a Credential Request for a credential in JWT VC format (JSON encoding) with a proof type `jwt`:

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/json
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

{
   "format":"jwt_vc_json",
   "types":[
      "VerifiableCredential",
      "UniversityDegreeCredential"
   ],
   "proof":{
      "proof_type":"jwt",
      "jwt":"eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8
      xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJzNkJoZFJrcXQzIiwiYXVkIjoiaHR
      0cHM6Ly9zZXJ2ZXIuZXhhbXBsZS5jb20iLCJpYXQiOiIyMDE4LTA5LTE0VDIxOjE5OjEwWiIsIm5vbm
      NlIjoidFppZ25zbkZicCJ9.ewdkIkPV50iOeBUqMXCC_aZKPxgihac0aW9EkL1nOzM"
   }
}
```
### Proof Types {#proof_types}

This specification defines the following values for `proof_type`:

* `jwt`: objects of this type contain a single `jwt` element with a JWS [@!RFC7515] as proof of possession. The JWT MUST contain the following elements:
    * in the JOSE Header,
        * `typ`: REQUIRED. MUST be `openid4vci-proof+jwt`, which explicitly types the proof JWT as recommended in Section 3.11 of [@!RFC8725].
        * `alg`: REQUIRED. A digital signature algorithm identifier such as per IANA "JSON Web Signature and Encryption Algorithms" registry. MUST NOT be `none` or an identifier for a symmetric algorithm (MAC).
        * `kid`: CONDITIONAL. JOSE Header containing the key ID. If the Credential shall be bound to a DID, the `kid` refers to a DID URL which identifies a particular key in the DID Document that the Credential shall be bound to. MUST NOT be present if `jwk` or `x5c` is present.
        * `jwk`: CONDITIONAL. JOSE Header containing the key material the new Credential shall be bound to. MUST NOT be present if `kid` or `x5c` is present.
        * `x5c`: CONDITIONAL. JOSE Header containing a certificate or certificate chain corresponding to the key used to sign the JWT. This element MAY be used to convey a key attestation. In such a case, the actual key certificate will contain attributes related to the key properties. MUST NOT be present if `kid` or `jwk` is present.
    * in the JWT body, 
        * `iss`: OPTIONAL (string). The value of this claim MUST be the `client_id` of the client making the credential request. This claim MUST be omitted if the Access Token authorizing the issuance call was obtained from a Pre-Authorized Code Flow through anonymous access to the Token Endpoint.
        * `aud`: REQUIRED (string). The value of this claim MUST be the Credential Issuer URL of the Credential Issuer.
        * `iat`: REQUIRED (number). The value of this claim MUST be the time at which the proof was issued using the syntax defined in [@!RFC7519].
        * `nonce`: REQUIRED (string). The value type of this claim MUST be a string, where the value is a `c_nonce` provided by the Credential Issuer.

The Credential Issuer MUST validate that the `proof` is actually signed by a key identified in the JOSE Header.

Below is a non-normative example of a `proof` parameter (line breaks for display purposes only):

```json
{
  "proof_type": "jwt",
  "jwt": "eyJraWQiOiJkaWQ6ZXhhbXBsZTplYmZlYjFmNzEyZWJjNmYxYzI3NmUxMmVjMjEva2V5cy8
  xIiwiYWxnIjoiRVMyNTYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJzNkJoZFJrcXQzIiwiYXVkIjoiaHR
  0cHM6Ly9zZXJ2ZXIuZXhhbXBsZS5jb20iLCJpYXQiOiIyMDE4LTA5LTE0VDIxOjE5OjEwWiIsIm5vbm
  NlIjoidFppZ25zbkZicCJ9.ewdkIkPV50iOeBUqMXCC_aZKPxgihac0aW9EkL1nOzM"
  }
```

where the JWT looks like this:

```json
{
  "alg": "ES256",
  "kid":"did:example:ebfeb1f712ebc6f1c276e12ec21/keys/1"
}.
{
  "iss": "s6BhdRkqt3",
  "aud": "https://server.example.com",
  "iat": 1659145924,
  "nonce": "tZignsnFbp"
}
```

Here is another example JWT not only proving possession of a private key but also providing key attestation data for that key:

```json
{
  "alg": "ES256",
  "x5c":[<key certificate + certificate chain for attestation>]
}.
{
  "iss": "s6BhdRkqt3",
  "aud": "https://server.example.com",
  "iat": 1659145924,
  "nonce": "tZignsnFbp"
}
```

## Credential Response {#credential-response}

Credential Response can be Synchronous or Deferred. The Credential Issuer MAY be able to immediately issue a requested Credential and send it to the Client. In other cases, the Credential Issuer MAY NOT be able to immediately issue a requested Credential and would want to send an `acceptance_token` parameter to the Client to be used later to receive a Credential when it is ready.

The following claims are used in the Credential Response:

* `format`: REQUIRED. JSON string denoting the format of the issued Credential.
* `credential`: OPTIONAL. Contains issued Credential. MUST be present when `acceptance_token` is not returned. MAY be a JSON string or a JSON object, depending on the Credential format. See (#format_profiles) for the Credential format specific encoding requirements.
* `acceptance_token`: OPTIONAL. A JSON string containing a security token subsequently used to obtain a Credential. MUST be present when `credential` is not returned.
* `c_nonce`: OPTIONAL. JSON string containing a nonce to be used to create a proof of possession of key material when requesting a Credential (see (#credential_request)). When received, the Wallet MUST use this nonce value for its subsequent credential requests until the Credential Issuer provides a fresh nonce.
* `c_nonce_expires_in`: OPTIONAL. JSON integer denoting the lifetime in seconds of the `c_nonce`.

The `format` Claim determines the Credential format and encoding of the credential in the Credential Response. Details are defined in the Credential Format Profiles in (#format_profiles). 

Credential formats expressed as binary data MUST be base64url-encoded and returned as a JSON string.

Below is a non-normative example of a Credential Response in a synchronous flow for a credential in JWT VC format (JSON encoded):

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "format": "jwt_vc_json",
  "credential" : "LUpixVCWJk0eOt4CXQe1NXK....WZwmhmn9OQp6YxX0a2L",
  "c_nonce": "fGFF7UkhLa",
  "c_nonce_expires_in": 86400  
}
```

Below is a non-normative example of a Credential Response in a deferred flow:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "acceptance_token": "8xLOxBtZp8",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400  
}
```

Note: Consider using CIBA Ping/Push OR SSE Poll/Push. Another option would be the Client providing `client_notification_token` to the Credential Issuer, so that the Credential Issuer sends a Credential response upon successfully receiving a Credential Request and then no need for the client to bring an acceptance token, the Credential Issuer will send the Credential once it is issued in a response that includes `client_notification_token`. (consider SSE options)

### Credential Error Response

When the Credential Request is invalid or unauthorized, the Credential Issuer constructs the error response as defined in this section.

The following additional clarifications are provided for the following parameters already defined in section 3.1 of [@!RFC6750]:

`invalid_request`:

- Credential Request was malformed. One or more of the parameters (i.e. `format`, `proof`) are missing or malformed.

`invalid_token`:

- Credential Request contains the wrong Access Token or the Access Token is missing

The following additional error codes are specified:

* `unsupported_credential_type`: requested credential type is not supported
* `unsupported_credential_format`:  requested credential format is not supported
* `invalid_or_missing_proof` - Credential Request did not contain a `proof`, or `proof` was invalid, i.e. it was not bound to a Credential Issuer provided nonce

This is a non-normative example of a Credential Error Response:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
   "error": "invalid_request"
}
```

### Credential Issuer Provided Nonce

Upon receiving a Credential Request, the Credential Issuer MAY require the client to send a proof of possession of the key material it wants a Credential to be bound to. This proof MUST incorporate a nonce generated by the Credential Issuer. The Credential Issuer will provide the client with a nonce in an error response to any Credential Request not including such a proof or including an invalid proof. 

Below is a non-normative example of a Credential Response when the Credential Issuer is requesting a Wallet to provide in a subsequent Credential Request a `proof` that is bound to a `c_nonce`:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store

{
  "error": "invalid_or_missing_proof",
  "error_description":
       "Credential Issuer requires proof to be bound to a Credential Issuer provided nonce.",
  "c_nonce": "8YE9hCnyV2",
  "c_nonce_expires_in": 86400  
}
```

# Batch Credential Endpoint {#batch-credential-endpoint}

The Batch Credential Endpoint issues multiple Credentials in one Batch Credential Response as approved by the End-User upon presentation of a valid Access Token representing this approval.

Communication with the Batch Credential Endpoint MUST utilize TLS. 

The client can request issuance of multiple Credentials of certain types and formats in one Batch Credential Request. This includes Credentials of the same type and multiple formats, different types and one format, or both. 

## Batch Credential Request {#batch-credential_request}

The Batch Credential Endpoint allows a client to send multiple Credential Request objects (see (#credential_request)) to request the issuance of multiple credential at once.

The following claims are used in the Batch Credential Request:

* `credential_requests`: REQUIRED. JSON array that contains Credential Request objects as defined in (#credential_request).

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
         "types":[
            "VerifiableCredential",
            "UniversityDegreeCredential"
         ],
         "proof":{
            "proof_type":"jwt",
            "jwt":"eyJraWQiOiJkaWQ6ZXhhbXBsZTpl...C_aZKPxgihac0aW9EkL1nOzM"
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

The following claims are used in the Batch Credential Response:

* `credential_responses`: REQUIRED. JSON array that contains Credential Response objects as defined in (#credential_request) and/or Deferred Credential Response objects as defined in (#deferred-credential_request). Every entry of the array corresponds to the Credential Request object at the same array index in the `credential_requests` parameter of the Batch Credential Request.
* `c_nonce`: OPTIONAL. The `c_nonce` as defined in (#credential-response). 
* `c_nonce_expires_in`: OPTIONAL. The `c_nonce_expires_in` as defined in (#credential-response). 

Below is a non-normative example of a Batch Credential Response in a synchronous flow:

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
  "credential_responses": [{
    "format": "jwt_vc_json",
    "credential" : "eyJraWQiOiJkaWQ6ZXhhbXBsZTpl...C_aZKPxgihac0aW9EkL1nOzM"
  },
  {
    "format": "mso_mdoc",
    "credential" : "YXNkZnNhZGZkamZqZGFza23....29tZTIzMjMyMzIzMjMy"
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
         "acceptance_token":"8xLOxBtZp8"
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

The Batch Credential Endpoint MUST respond with an HTTP status code 4xx in case of an error. 

The Batch Credential Request either succeessfully issues all requested credentials or fails entirely if there is even one credential failed to be issued.

# Deferred Credential Endpoint {#deferred-credential-issuance}

This endpoint is used to issue a Credential previously requested at the Credential Endpoint or Batch Credential Endpoint in case the Credential Issuer was not able to immediately issue this Credential. 

## Deferred Credential Request {#deferred-credential_request}

This is an HTTP POST request, which accepts an `acceptance_token` as the only parameter. The `acceptance_token` parameter MUST be sent as Access Token in the HTTP header as shown in the following example.

```
POST /credential_deferred HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: BEARER 8xLOxBtZp8

```

## Deferred Credential Response {#deferred-credential_response}

The deferred Credential Response uses the `format` and `credential` parameters as defined in (#credential-response). 

# Metadata

## Client Metadata {#client-metadata}

This specification defines the following new Client Metadata parameter in addition to [@!RFC7591] for Wallets acting as OAuth 2.0 client:

* `credential_offer_endpoint`: OPTIONAL. URL of the Credential Offer Endpoint of a Wallet. 

If the Credential Issuer is unable to perform discovery of the Credential Offer Endpoint URL, the following claimed URL is used: `openid-credential-offer://`.

## Credential Issuer Metadata {#credential-issuer-metadata}

### Credential Issuer Identifier {#credential-issuer-identifier}

A Credential Issuer is identified by a case sensitive URL using the `https` scheme that contains scheme, host and, optionally, port number and path components, but no query or fragment components. 

### Credential Issuer Metadata Retrieval  {#credential-issuer-wellknown}

The Credential Issuer's configuration can be retrieved using the Credential Issuer Identifier.

Credential Issuers publishing metadata MUST make a JSON document available at the path formed by concatenating the string `/.well-known/openid-credential-issuer` to the Credential Issuer Identifier. If the Credential Issuer value contains a path component, any terminating `/` MUST be removed before appending `/.well-known/openid-credential-issuer`. 

`openid-credential-issuer` MUST point to a JSON document compliant with this specification and MUST be returned using the `application/json` content type.

### Credential Issuer Metadata Parameters

This specification defines the following Credential Issuer Metadata:

* `credential_issuer`: REQUIRED. The Credential Issuer's identifier.
* `authorization_server`: OPTIONAL. Identifier of the OAuth 2.0 Authorization Server (as defined in [@!RFC8414]) the Credential Issuer relies on for authorization. If this element is omitted, the entity providing the Credential Issuer is also acting as the AS, i.e. the Credential Issuer's identifier is used as the OAuth 2.0 Issuer value to obtain the Authorization Server metadata as per [@!RFC8414]. 
* `credential_endpoint`: REQUIRED. URL of the Credential Issuer's Credential Endpoint. This URL MUST use the `https` scheme and MAY contain port, path and query parameter components.
* `batch_credential_endpoint`: OPTIONAL. URL of the Credential Issuer's Batch Credential Endpoint. This URL MUST use the `https` scheme and MAY contain port, path and query parameter components. If omitted, the Credential Issuer does not support the Batch Credential Endpoint.

The following parameter MUST be used to communicate the specifics of the Credential that the Credential Issuer supports issuance of:

* `credentials_supported`: REQUIRED. A JSON array containing a list of JSON objects, each of them representing metadata about a separate credential type that the Credential Issuer can issue. The JSON objects in the array MUST conform to the structure of the (#credential-metadata-object). 

* `display`: OPTIONAL. An array of objects, where each object contains display properties of a Credential Issuer for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
  * `name`: OPTIONAL. String value of a display name for the Credential Issuer.
  * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. There MUST be only one object with the same language identifier

#### Objects comprising `credentials_supported` parameter{#credential-metadata-object}

This section defines the structure of the objects that appear in the `credentials_supported` metadata parameter.

* `format`: REQUIRED. A JSON string identifying the format of this credential, e.g. `jwt_vc_json` or `ldp_vc`. Depending on the format value, the object contains further elements defining the type and (optionally) particular claims the credential MAY contain, and information how to display the credential. (#format_profiles) defines Credential Format Profiles introduced by this specification. 
* `id`: OPTIONAL. A JSON string identifying the respective object. The value MUST be unique across all `credentials_supported` entries in the Credential Issuer Metadata.
* `cryptographic_binding_methods_supported`: OPTIONAL. Array of case sensitive strings that identify how the Credential is bound to the identifier of the End-User who possesses the Credential as defined in (#credential-binding). Support for keys in JWK format [@!RFC7517] is indicated by the value `jwk`. Support for keys expressed as a COSE Key object [@!RFC8152] (for example, used in [@!ISO.18013-5]) is indicated by the value `cose_key`. When Cryptographic Binding Method is a DID, valid values MUST be a `did:` prefix followed by a method-name using a syntax as defined in Section 3.1 of [@!DID-Core], but without a `:`and method-specific-id. For example, support for the DID method with a method-name "example" would be represented by `did:example`. Support for all DID methods listed in Section 13 of [@DID_Specification_Registries] is indicated by sending a DID without any method-name.

* `cryptographic_suites_supported`: OPTIONAL. Array of case sensitive strings that identify the cryptographic suites that are supported for the `cryptographic_binding_methods_supported`. Cryptosuites for Credentials in `jwt_vc` format should use algorithm names defined in [IANA JOSE Algorithms Registry](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms). Cryptosuites for Credentials in `ldp_vc` format should use signature suites names defined in [Linked Data Cryptographic Suite Registry](https://w3c-ccg.github.io/ld-cryptosuite-registry/).
* `display`: OPTIONAL. An array of objects, where each object contains the display properties of the supported credential for a certain language. Below is a non-exhaustive list of parameters that MAY be included. Note that the display name of the supported credential is obtained from `display.name` and individual claim names from `claims.display.name` values.
    * `name`: REQUIRED. String value of a display name for the Credential.
    * `locale`: OPTIONAL. String value that identifies the language of this object represented as a language tag taken from values defined in BCP47 [@!RFC5646]. Multiple `display` objects MAY be included for separate languages. There MUST be only one object with the same language identifier.
    * `logo`: OPTIONAL. A JSON object with information about the logo of the Credential with a following non-exhaustive list of parameters that MAY be included:
        * `url`: OPTIONAL. URL where the Wallet can obtain a logo of the Credential from the Credential Issuer.
        * `alt_text`: OPTIONAL. String value of an alternative text of a logo image.
    * `description`: OPTIONAL. String value of a description of the Credential.
    * `background_color`: OPTIONAL. String value of a background color of the Credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].
    * `text_color`: OPTIONAL. String value of a text color of the Credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].

It is dependent on the Credential format where the available claims will appear and how they are represented (see (#format_profiles)).

The following example shows a non-normative example of an object comprising `credentials_supported` parameter for a credential in JWT VC format (JSON encoding).

<{{examples/credential_metadata_jwt_vc_json.json}}

Note: The Client MAY use other mechanisms to obtain information about the Verifiable Credentials that a Credential Issuer can issue.

## OAuth 2.0 Authorization Server Metadata

This specification also defines a new OAuth 2.0 Authorization Server metadata [@!RFC8414] parameter to publish whether the AS that the Credential Issuer relies on for authorization, supports anonymous Token Requests with the Pre-authorized Grant Type. It is defined as follows:

* `pre-authorized_grant_anonymous_access_supported`: OPTIONAL. A JSON Boolean indicating whether the issuer accepts a Token Request with a Pre-Authorized Code but without a client id. The default is `false`. 

# Security Considerations {#security-considerations}

## Trust between Wallet and Issuer

Credential Issuers often want to know what Wallet they are issuing Credentials to and how private keys are managed for the following reasons:

* the Credential Issuer MAY want to ensure that private keys are properly protected from exfiltration and replay in order to prevent an adversary from impersonating the legitimate Credential holder by presenting her Credential.
* the Credential Issuer MAY also want to ensure that the Wallet managing the Credentials adheres to certain policies and, potentially, was audited and approved under a certain regulatory and/or commercial scheme. 

The following mechanisms in concert can be utilized to fulfill those objectives:

**Key attestation** is a mechanism where the device or security element in a device asserts the key management policy to the application creating and using this key. The Android Operating System, for example, provides apps with a certificate including a certificate chain asserting that a particular key is managed, for example, by a hardware security module [ref]. The Wallet can provide this data along with the proof of possession in the Credential Request (see (#credential_request) for an example) to allow the Credential Issuer to validate the key management policy. This indeed requires the Credential Issuer to rely on the trust anchor of the certificate chain and the respective key management policy. Another variant of this concept is the use of a Qualified Electronic Signature as defined by the eIDAS regulation [ref]. This signatures won't reveal the concrete properties of the associated private key to the Credential Issuer. However, due to the regulatory regime of eIDAS the Credential Issuer can deduce that the signing service manages the private keys according to this regime and fulfills very high security requirements. As another example, FIDO2 allows RPs to obtain an attestation along with the public key from a FIDO authenticator. That implicitly asserts the key management policy since the assertion is bound to a certain authenticator model and its key management capabilities. 

**App Attestation**: Key attestation, however, does not establish trust in the application storing the Credential and producing presentation of that Credential. App attestation as provided by mobile operating systems, e.g. iOS's DeviceCheck or Android's Safetynet, allows a server system to ensure it is communicating to a legitimate instance of its genuine app. Those mechanisms can be utilized to validate the internal integrity of the Wallet (as a whole).  

**Device Attestation**: Device Attestation attests the health of the device, on which the Wallet is running, as a whole. It prevents compromises such as a malicious third party application tampering with a Wallet that manages keys and Credentials, which cannot be captured only by obtaining app attestation of a Wallet.

**Client Authentication** allows a Wallet to authenticate with a Credential Issuer. In order to securely authenticate, the Wallet needs to utilize a backend component managing the key material and processing the secure communication with the Credential Issuer. The Credential Issuer MAY establish trust with the Wallet based on its own auditing or a trusted 3rd party attestation of the conformance of the Wallet to a certain policy.  

Directly using key, app and/or device attestations to prove certain capabilities towards a Credential Issuer is an obvious option. However, this at least requires platform mechanisms that issue signed assertions that 3rd parties can evaluate, which is not always the case (e.g. iOS's DeviceCheck). Also, such an approach creates dependencies on platform specific mechanisms, trust anchors, and platform specific identifiers (e.g. Android `apkPackageName`) and it reveals information about the internal design of a Wallet. Implementers should take these consequences into account. 

The approach recommended by this specification is that the Credential Issuer relies on the OAuth 2.0 Client Authentication to establish trust in the Wallet and leaves it to the Wallet to ensure its internal integrity using app and key attestation (if required). This establishes a clean separation between the different hemispheres and a uniform interface irrespectively of the Wallet's architecture (e.g. native vs web Wallet). Client Authentication can be performed with assertions registered with the Credential Issuer or with assertions issued to the Wallet by a 3rd party the Credential Issuer trusts for the purpose of Client Authentication.  

## Credential Offer {#credential-offer-security}

The Wallet MUST consider the parameter values in the Credential Offer as not trustworthy since the origin is not authenticated and the message integrity is not protected. The Wallet MUST apply the same checks on the Credential Issuer that it would apply when the flow is started from the Wallet itself since the Credential Issuer is not trustworthy just because it sent the Credential Offer. An attacker might attempt to use a Credential Offer to conduct a phishing or injection attack. 

The Wallet MUST NOT accept Credentials just because this mechanism was used. All protocol steps defined in this specification MUST be performed in the same way as if the Wallet would have started the flow. 

The Credential Issuer MUST ensure the release of any privacy-sensitive data in Credential Offer is legally based.

## Pre-Authorized Code Flow {#security_considerations_pre-authz-code}

### Replay Prevention

The Pre-Authorized Code Flow is vulnerable to the replay of the Pre-Authorized Code, because by design it is not bound to a certain device (as the Authorization Code Flow does with PKCE). This means an attacker can replay at another device the Pre-Authorized Code meant for a victim, e.g., the attacker can scan the QR code while it is displayed on the victim’s screen, and thereby get access to the Credential. Such replay attacks must be prevented using other means. The design facilitates the following options: 

* User PIN: the Credential Issuer might set up a PIN with the End-User (e.g. via text message or email), which needs to be presented in the Token Request.
* Callback to device where the transaction originated: upon receiving the Token Request, the Credential Issuer asks the End-User to confirm the originating device (device that displayed the QR code) that the Credential Issuer MAY proceed with the Credential issuance process. While the Credential Issuer reaches out to the End-User on the other device to get confirmation, the Credential Issuer returns an `authorization_pending` error code to the Wallet as described in (#token-response). The Wallet is required to call the Token Endpoint again to obtain the Access Token. If the End-User does not confirm, the Token Request is returned with the `access_denied` error code. This flow gives the End-User on the originating device more control over the issuance process.

### PIN Code Phishing

An attacker might leverage the Credential issuance process and the user's trust into the Wallet to phish PIN codes sent out by a different service that grant attacker access to services other than Credential issuance. The attacker could setup a Credential Issuer site and in parallel to the issuance request trigger transmission of a PIN code to the user's phone from a service other than Credential issuance, e.g. from a payment service. The user would then be asked to enter this PIN into the Wallet and since the Wallet sends this PIN to the Token Endpoint of the Credential Issuer (the attacker), the attacker would get access to the PIN code, and access to that other service. 

In order to cope with that issue, the Wallet is RECOMMENDED to interact with trusted Credential Issuers only. In that case, the Wallet would not process a Credential Offer with an untrusted issuer URL. The Wallet MAY also show to the End-User the endpoint of the Credential Issuer it will be sending the PIN code to and ask the End-User for confirmation.

## Credential Lifecycle Management 

the Credential Issuer is supposed to be responsible for the lifecycle of its Credentials. This means the Credential Issuer will invalidate Credentials if it deems appropriate, e.g. if it detects fraudulent behavior.

The Wallet is supposed to detect signs of fraudulant behavior related to the Credential management in the Wallet (e.g. device rooting) and to act upon such signals. Options include Credential revocation at the Credential Issuer and/or invalidation of the key material used to cryptographically bind Credential to the identifier of the End-User possessing that Credential.  

# Implementation Considerations

## Claim-based Binding of the Credential to the End-User possessing the Credential {#claim-based-binding}

Credential not cryptographically bound to the identifier of the End-User possessing it (see (#credential-binding)), should be bound to the End-User possessing the Credential based on the claims included in that Credential. 

In claim-based binding, no cryptographic binding material is provided. Instead, the issued Credential includes user claims that can be used by the Verifier to verify possession of the Credential by requesting presentation of existing forms of physical or digial identification that includes the same claims (e.g., a driver's license or other ID cards in person, or an online ID verification service).

## Binding of the Credential without Cryptographic Binding nor Claim-based Binding {#no-binding}

Some Credential Issuers might choose issuing bearer Credentials without either cryptographic binding nor claim-based binding, because they are meant to be presented without proof of possession.

One such use case is low assurance Credentials such as coupons or tickets. 

Another use case is when the Credential Issuer uses cryptographic schemes that can provide binding to the End-User possessing that Credential without explicit cryptographic material being supplied by the application used by that End-User. For example, in the case of the BBS Signature Scheme, the issued Credential itself is a secret and only derivation of a Credential is presented to the Verifier. Effectively, Credential is bound to the Credential Issuer's signature on the Credential, which becomes a shared secret transferred from the Credential Issuer to the End-User.

## Multiple Accesses to the Credential Endpoint

The Credential Endpoint can be accessed multiple times by a Wallet using the same Access Token, even for the same Credential. The Credential Issuer determines if the subsequent successful requests will return the same or an updated Credential, such as having a new expiration time or using the most current End-User claims.

the Credential Issuer MAY also decide that the current Access Token is no longer be valid and a re-authentication or Token Refresh (see [@!RFC6749, section 6]) MAY be required under the Credential Issuer's discretion. The policies between the Credential Endpoint and the Authorization Server that MAY change the behavior of what is returned with a new Access Token are beyond the scope of this specification (see [@!RFC6749, section 7]).

The action leading to the Wallet performing another Credential Request can also be triggered by a background process, or by the Credential Issuer using an out-of-band mechanism (SMS, email, etc.) to inform the End-User.

## Relationship between the Credential Issuer Identifier in the metadata and the Issuer Identifier in the Issued Credential

Credential Issuer Identifier is always a URL using the `https` scheme as defined in (#credential-issuer-identifier). Depending on the Credential format, the issuer identifier in the issued Credential is not always a URL using the `https` scheme. Some other forms that it can take are a DID included in the `issuer` property in a [@VC_DATA] format, or the `Subject` value of the document signer certificate included in the `x5chain` element in a [@ISO.18013-5] format.

When the issuer identifier in the issued Credential is a DID, below is a non-exhaustive list of mechanisms how Credential Issuer MAY provide binding to the Credential Issuer Identifier:

1. Use [@DIF.Well-Known_DID] Specification to provide binding between a DID and a certain domain.
1. If the issuer identifier in the issued Credential is an object, add to the object `credential_issuer` Claim as defined in {#credential-issuer-identifier}.

The Wallet MAY check the binding between Credential Issuer Identifier and the issuer identifier in the issued Credential.

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

<reference anchor="RFC6750" target="https://www.rfc-editor.org/rfc/rfc6750">
  <front>
    <title>The OAuth 2.0 Authorization Framework: Bearer Token Usage</title>
    <author fullname="Dick Hardt">
      <organization>Independent</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
   <date month="October" year="2012"/>
  </front>
</reference>

<reference anchor="SIOPv2" target="https://openid.net/specs/openid-connect-self-issued-v2-1_0.html">
  <front>
    <title>Self-Issued OpenID Provider V2</title>
    <author ullname="Kristina Yasuda">
      <organization>Microsoft</organization>
    </author>
    <author fullname="Michael B. Jones">
      <organization>Microsoft</organization>
    </author>
   <date day="18" month="December" year="2021"/>
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

<reference anchor="OpenID4VP" target="https://openid.net/specs/openid-4-verifiable-presentations-1_0.html">
      <front>
        <title>OpenID for Verifiable Presentations</title>
        <author initials="O." surname="Terbu" fullname="Oliver Terbu">
         <organization>ConsenSys Mesh</organization>
        </author>
        <author initials="T." surname="Lodderstedt" fullname="Torsten Lodderstedt">
          <organization>yes.com</organization>
        </author>
        <author initials="K." surname="Yasuda" fullname="Kristina Yasuda">
          <organization>Microsoft</organization>
        </author>
        <author initials="A." surname="Lemmon" fullname="Adam Lemmon">
          <organization>Convergence.tech</organization>
        </author>
        <author initials="T." surname="Looker" fullname="Tobias Looker">
          <organization>Mattr</organization>
        </author>
       <date day="20" month="June" year="2022"/>
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

# IANA Considerations

## Sub-Namespace Registration

This section registers the value "urn:ietf:params:oauth:grant-type:pre-authorized_code" in the IANA "OAuth URI" registry established by "An IETF URN Sub-Namespace for OAuth" [@!RFC6755].

* URN: urn:ietf:params:oauth:grant-type:pre-authorized_code
* Common Name: Pre-Authorized Code
* Change Controller: AB/Connect Working Group - openid-specs-ab@lists.openid.net
* Specification Document: (#token_request) of this document

## Well-Known URI Registry

This specification registers the well-known URI defined in (#credential-issuer-wellknown) in the IANA Well-Known URI registry defined in RFC 5785 [@!RFC5785].

* URI suffix: openid-credential-issuer
* Change controller: AB/Connect Working Group - openid-specs-ab@lists.openid.net
* Specification document: (#credential-issuer-wellknown) of this document
* Related information: (none)

# Acknowledgements {#Acknowledgements}

We would like to thank Daniel Fett, Brian Campbell, Joseph Heenan, Daniel McGrogan, David Chadwick, John Bradley, Mark Haine, Alen Horvat, Michael B. Jones, Kenichi Nakamura, Oliver Terbu and David Waite for their valuable contributions to this specification.

# Notices

Copyright (c) 2022 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that MAY cover technology that MAY be required to practice this specification.

# Use Cases

This is a non-exhaustive list of sample use cases.

## Credential Offer - Same-Device {#use-case-3}

While browsing the university's home page, the End-User finds a link "request your digital diploma". User  clicks on this link and is being redirected to a digital Wallet. The Wallet notifies the End-User that a Credential Issuer offered to issue a diploma Credential. User confirms this inquiry and is taken to the university's Credential issuance service's user experience. After authenticating at the university and consenting to the issuance of a digital diploma, the End-User is sent back to the Wallet, where she can check the successful creation of the digital diploma.

## Credential Offer - Cross-Device (with information pre-submitted by the End-User) {#use-case-4}

The End-User is starting a job at a new employer. An employer has requested the End-User to upload certain documents to the employee portal. A few days later, the End-User receives an email from the employer notifying her that the employee Credential is ready and asking her to scan a QR code to retrieve it. User scans the QR code with her smartphone, which opens her Wallet. Meanwhile, the End-User has received a text message with a PIN code to her smartphone. After entering that PIN code in the Wallet for security reasons, the End-User confirms the Credential issuance, and receives Credential into the Wallet.

## Credential Offer - Cross-Device & Deferred {#use-case-5}

The End-User wants to obtain a digital criminal record. She visits the local administration's office and requests the issuance of the official criminal record as a digital Credential. After presenting her ID document, she is asked to scan a QR code with her Wallet. She is being told that the actual issuance of the Credential will take some time due to necessary background checks by the authority. 

In the Wallet, the End-User sees an indication that issuance of the digital record is under way. A few days later, the End-User receives a notification from her Wallet that requested Credential was successfully issued. When the End-User opens the Wallet, she is asked whether she wants to download the Credential. She confirms, and the new Credential is retrieved and stored in the Wallet.

## Wallet Initiated Issuance during Presentation {#use-case-1}

An End-User comes across a verifier app that is requesting the End-User to present a Credential, e.g., a driving license. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet determines a Credential Issuer capable of issuing the lacking Credential and upon user consent sends the End-User to the Credential Issuer's user experience (web site or app). Upon being authenticated and providing consent to issue the Credential into her Wallet, the End-User is sent back to the Wallet. The Wallet informs the End-User that Credential was successfully issued into the Wallet and is ready to be presented to the verifier app that originally requested presentation of that Credential.

## Wallet Initiated Issuance during Presentation (requires presentation of additional Credentials during issuance) {#use-case-2}

An End-User comes across a verifier app that is requesting the End-User to present a Credential, e.g., a university diploma. The Wallet determines the requested Credential type(s) from the presentation request and notifies the End-User that there is currently no matching Credential in the Wallet. The Wallet then offers the End-User a list of Credential Issuers, which might be based on a Credential Issuer list curated by the Wallet provider. The End-User picks the university she graduated from and is sent to that university's user experience (web site or app).  

The End-User logs in to the university, which determines that the respective user account is not verified yet. Among multiple identification options, the End-User chooses to present identity Credential from her Wallet. The End-User is sent back to the Wallet where she consents to share requested Credential(s) to the university. The End-User is sent back to the university user experience. Based on the presented Credential, the university completes the user verification, looks up user data in its database, and offers to issue a diploma as a verifiable Credential. 

Upon providing consent, the End-User is sent back to the Wallet. The Wallet informs the user Credential was successfully issued into the Wallet and is ready to be presented to the verifier app that originally requested presentation of that Credential.

# Credential Format Profiles {#format_profiles}

This specification defines several extension points to accommodate the differences across Credential formats. Sets of Credential format specific parameters or claims referred to as Credential format Identifiers are identified by the Credential format identifier and used at each extension point.

This section defines Credential Format Profiles for selected Credential formats.

## W3C Verifiable Credentials

Sections 6.1 and 6.2 of [@VC_DATA] defines how Verifiable Credentials MAY or MAY NOT use JSON-LD. As acknowleged in Sections 4.1 of [@VC_DATA], implementations can behave differently regarding processing of the `@context` property whether JSON-LD is used or not.

This specification therefore differentiates the following three Credential formats for W3C Verifiable Credentials:

* VC signed as a JWT, not using JSON-LD (`jwt_vc_json`)
* VC signed as a JWT, using JSON-LD (`jwt_vc_json-ld`)
* VC secured using Data Integrity, using JSON-LD, with proof suite requiring Linked Data canonicalization (`ldp_vc`)

Note that VCs secured using Data Integrity MAY NOT necessarily use JSON-LD and MAY NOT necessarily use proof suites requiring Linked Data canonicalization. Credential Format Profiles for them MAY be defined in the future versions of this specification.

Distinct Credential formats identifiers, extension parameters/claims and processing rules are defined for each of the above-mentioned Credential formats.

### VC signed as a JWT, not using JSON-LD

#### Format Identifier

The Credential format identifier is `jwt_vc_json`.

#### Credential Issuer Metadata {#server_metadata_jwt_vc_json}

The following additional Credential Issuer metadata are defined for this Credential format. 

* `types`: REQUIRED. JSON array designating the types a certain credential type supports according to [@VC_DATA], Section 4.3.
* `credentialSubject`: OPTIONAL. A JSON object containing a list of key value pairs, where the key identifies the claim offered in the Credential. The value MAY be a dictionary, which allows to represent the full (potentially deeply nested) structure of the verifiable credential to be issued. The value is a JSON object detailing the specifics about the support for the claim with a following non-exhaustive list of parameters that MAY be included:
    * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued Credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
    * `value_type`: OPTIONAL. String value determining type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
    * `display`: OPTIONAL. An array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object with the same language identifier.
* `order`: OPTIONAL. An array of claims.display.name values that lists them in the order they should be displayed by the Wallet.

The following is a non-normative example of an object comprising `credentials_supported` parameter of Credential format `jwt_vc_json`.

<{{examples/credential_metadata_jwt_vc_json.json}}

#### Credential Offer

The following additional claims are defined for this Credential format. 

* `types`: REQUIRED. JSON array as defined in (#server_metadata_jwt_vc_json). This claim contains the type values the Wallet shall request in the subsequent Credential Request.

The following is a non-normative example of an object comprising `credentials_supported` parameter of Credential format `jwt_vc_json`.

<{{examples/credential_offer_jwt_vc_json.json}}

#### Authorization Details {#authorization_jwt_vc_json}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.

* `types`: REQUIRED. JSON array as defined in (#server_metadata_jwt_vc_json). This claim contains the type values the Wallet requests authorization for at the issuer.
* `credentialSubject`: OPTIONAL. A JSON object containing a list of key value pairs, where the key identifies the claim offered in the Credential. The value MAY be a dictionary, which allows to represent the full (potentially deeply nested) structure of the verifiable credential to be issued. This object indicates the claims the Wallet would like to turn up in the credential to be issued. 

The following is a non-normative example of an authorization details object with Credential format `jwt_vc_json`.

<{{examples/authorization_details_jwt_vc_json.json}}

#### Credential Request

The following additional parameters are defined for Credential Requests and this Credential format.  

* `types`: REQUIRED. JSON array as defined in (#authorization_jwt_vc_json). The credential issued by the issuer MUST at least contain the values listed in this claim. 
* `credentialSubject`: OPTIONAL. A JSON object as defined in (#authorization_jwt_vc_json). This object determines the optional claims to be added to the credential to be issued. 

The following is a non-normative example of a Credential Request with Credential format `jwt_vc_json`.

<{{examples/credential_request_jwt_vc_json_with_claims.json}}

#### Credential Response {#credential_response_jwt_vc_json}

The value of the `credential` claim in the Credential Response MUST be a JSON string. Credentials of this format are already a sequence of base64url-encoded values separated by period characters and MUST NOT be re-encoded. 

The following is a non-normative example of a Credential Response with Credential format `jwt_vc_json`.

<{{examples/credential_response_jwt_vc_json.txt}}

### VC signed as a JWT, using JSON-LD

#### Format Identifier

The Credential format identifier is `jwt_vc_json-ld`.

#### Credential Issuer Metadata

The definitions in (#server_metadata_ldp_vc) apply for metadata of credentials of this type as well. 

#### Credential Offer

The definitions in (#issuer_initiated_issuance_ldp_vc) apply for credentials of this type as well. 

#### Authorization Details

The definitions in (#issuer_initiated_issuance_ldp_vc) apply for credentials of this type as well.

#### Credential Request

The definitions in (#credential_request_ldp_vc) apply for credentials of this type as well.

#### Credential Response

The definitions in (#credential_response_jwt_vc_json) apply for credentials of this type as well.

### VC secured using Data Integrity, using JSON-LD, with proof suite requiring Linked Data canonicalization

#### Format Identifier

The Credential format identifier is `ldp_vc`.

Note that Data Integrity used to be called Linked Data Proofs, hence "ldp" in the Credential format identifier.

#### Credential Issuer Metadata {#server_metadata_ldp_vc}

The following additional Credential Issuer metadata are defined for this Credential format. 

* `@context`: REQUIRED. JSON array as defined in [@VC_DATA], Section 4.1.
* `types`: REQUIRED. JSON array designating the types a certain credential type supports according to [@VC_DATA], Section 4.3.
* `credentialSubject`: OPTIONAL. A JSON object containing a list of key value pairs, where the key identifies the claim offered in the Credential. The value MAY be a dictionary, which allows to represent the full (potentially deeply nested) structure of the verifiable credential to be issued. The value is a JSON object detailing the specifics about the support for the claim with a following non-exhaustive list of parameters that MAY be included:
    * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued Credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
    * `value_type`: OPTIONAL. String value determining type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
    * `display`: OPTIONAL. An array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object with the same language identifier.
* `order`: OPTIONAL. An array of claims.display.name values that lists them in the order they should be displayed by the Wallet.

Any object comprising `credentials_supported` parameter of Credential format `ldp_vc` MUST be processed using full JSON-LD processing.

The following is a non-normative example of an object comprising `credentials_supported` parameter of Credential format `ldp_vc`.

<{{examples/credential_metadata_ldp_vc.json}}

#### Credential Offer {#issuer_initiated_issuance_ldp_vc}

The following additional claims are defined for this Credential format. 

* `credential_definition`: REQUIRED. JSON object containing (and isolating) the detailed description of the credential type. This object MUST be processed using full JSON-LD processing. It consists of the following sub claims:
    * `@context`: REQUIRED. JSON array as defined in (#server_metadata_ldp_vc)
    * `types`: REQUIRED. JSON array as defined in (#server_metadata_ldp_vc). This claim contains the type values the Wallet shall request in the subsequent Credential Request. 

The following is a non-normative example of a Credential Offer of type `ldp_vc`.

<{{examples/credential_offer_ldp_vc.json}}

#### Authorization Details {#authorization_ldp_vc}

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.  

* `credential_definition`: REQUIRED. JSON object as defined in (#issuer_initiated_issuance_ldp_vc).

The following is a non-normative example of an authorization details object with Credential format `ldp_vc`.

<{{examples/authorization_details_ldp_vc.json}}

#### Credential Request {#credential_request_ldp_vc}

The following additional parameters are defined for Credential Requests and this Credential format.  

* `credential_definition`: REQUIRED. JSON object as defined in (#issuer_initiated_issuance_ldp_vc).

The following is a non-normative example of a Credential Request with Credential format `ldp_vc`.

<{{examples/credential_request_ldp_vc.json}}

#### Credential Response

The value of the `credential` claim in the Credential Response MUST be a JSON object. Credentials of this format MUST NOT be re-encoded.

The following is a non-normative example of a Credential Response with Credential format `ldp_vc`.

<{{examples/credential_response_ldp_vc.txt}}

## ISO mDL

This section defines a Credential Format Profile for credentials complying with [@!ISO.18013-5].

### Format Identifier

The Credential format identifier is `mso_mdoc`.

### Credential Issuer Metadata {#server_metadata_mso_mdoc}

The following additional Credential Issuer metadata are defined for this Credential format. 

* `doctype`: REQUIRED. JSON string identifying the credential type. 
* `claims`: OPTIONAL. A JSON object containing a list of key value pairs, where the key is a certain `namespace` as defined in [@!ISO.18013-5] (or any profile of it), and the value is a JSON object. This object also contains a list of key value pairs, where the key is a claim that is defined in the respective namespace and is offered in the Credential. The value is a JSON object detailing the specifics of the claim with a following non-exhaustive list of parameters that MAY be included:
    * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued Credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
    * `value_type`: OPTIONAL. String value determining type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
    * `display`: OPTIONAL. An array of objects, where each object contains display properties of a certain claim in the Credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
        * `name`: OPTIONAL. String value of a display name for the claim.
        * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object with the same language identifier.
* `order`: OPTIONAL. An array of claims.display.name values that lists them in the order they should be displayed by the Wallet.

The following is a non-normative example of an object comprising `credentials_supported` parameter of Credential format `mso_mdoc`.

<{{examples/credential_metadata_mso_mdoc.json}}

### Credential Offer

The following additional claims are defined for this Credential format. 

* `doctype`: REQUIRED. JSON string as defined in (#server_metadata_mso_mdoc) 

The following is a non-normative example of a Credential Offer of type `mso_mdoc`.

<{{examples/credential_offer_mso_doc.json}}

### Authorization Details

The following additional claims are defined for authorization details of type `openid_credential` and this Credential format.

* `doctype`: REQUIRED. JSON string as defined in (#server_metadata_mso_mdoc) 
* `claims`: OPTIONAL. A JSON object as defined in (#server_metadata_mso_mdoc)  

The following is a non-normative example of an authorization details object with Credential format `mso_mdoc`.

<{{examples/authorization_details_mso_doc.json}}

### Credential Request

The following additional parameters are defined for Credential Requests and this Credential format.  

* `doctype`: REQUIRED. JSON string as defined in (#server_metadata_mso_mdoc) 
* `claims`: OPTIONAL. A JSON object as defined in (#server_metadata_mso_mdoc) 

The following is a non-normative example of a Credential Request with Credential format `mso_mdoc`.

<{{examples/credential_request_iso_mdl_with_claims.json}}

### Credential Response

The value of the `credential` claim in the Credential Response MUST be a JSON string that is the base64url-encoded representation of the issued Credential.

# Document History

   [[ To be removed from the final specification ]]

   -10

   * introduced differentiation between Credential Issuer and Authorization Server 
   * relaxed client identification requirements for Pre-Authorized Code Grant Type
   * renamed issuance initiation endpoint to Credential Offer Endpoint
   * added `grants` structure to credential offer

   -09

   * reworked credential type identification and issuer metadata
   * changed format of issuer initiated Credential Request to JSON
   * added option to include credential data by reference in issuer initiated Credential Request
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
