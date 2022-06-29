%%%
title = "OpenID for Verifiable Credential Issuance"
abbrev = "openid-4-verifiable-credential-issuance"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-4-verifiable-credential-issuance-1_0-06"
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

This specification defines an API and corresponding OAuth 2.0-based authorization mechanisms for issuance of verifiable credentials. This specification can be combined with OpenID Connect to obtain identity assertions along with verifiable credentials. 

{mainmatter}

# Introduction

This specification defines an API designated as Credential Endpoint and corresponding OAuth 2.0-based authorization mechanisms for issuance of verifiable credentials, e.g., in the form of W3C Verifiable Credentials. This allows existing OAuth 2.0 deployments and OpenID Connect OPs to extend their service and become credential issuers. It also allows new applications built using Verifiable Credentials to utilize OAuth 2.0 and OpenID Connect as integration and interoperability layer.

OpenID Connect would be an obvious choice for this use case since it already allows Relying Parties to request identity assertions. However, early implementation experience suggests adoption is made easier if the base protocol only focuses on the credential issuance, without mandating certain [@OpenID.Core] features, such as asserting a stable `sub` values. Thus the working group decided to use OAuth 2.0 as a base protocol. Deployments can nevertheless provide a combined implementation of OpenID Connect and the Credential Endpoint since OpenID Connect is built on top of OAuth 2.0. 

Verifiable Credentials are very similar to identity assertions in that they allow an Issuer to assert End-User claims. However, in contrast to the identity assertions, a verifiable credential follows a pre-defined schema (the credential type) and is bound to key material allowing the End-User to prove the legitimate possession of the credential. This allows direct presentation of the credential from the End-User to the RP, without involvement of the credential issuer. This specification caters for those differences.

# Terminology

Credential

A set of one or more claims made by an issuer (see [@VC_DATA]). Note that this definition differs from that in [@OpenID.Core].

Verifiable Credential (VC)

A verifiable credential is a tamper-evident credential that has authorship that can be cryptographically verified. Verifiable credentials can be used to build verifiable presentations, which can also be cryptographically verified (see [@VC_DATA]).

Presentation

Data derived from one or more verifiable credentials, issued by one or more issuers, that is shared with a specific verifier (see [@VC_DATA]).

Verified Presentation (VP)

A verifiable presentation is a tamper-evident presentation encoded in such a way that authorship of the data can be trusted after a process of cryptographic verification. Certain types of verifiable presentations might contain data that is synthesized from, but do not contain, the original verifiable credentials (for example, zero-knowledge proofs) (see [@VC_DATA]).

Deferred Credential Issuance

Issuance of credentials not directly in the response to a credential issuance request, but following a period of time that can be used to perform certain offline business processes.

Wallet

Entity that receives, stores, presents, and manages credentials and key material of the End-User. There is no single deployment model of a Wallet: credentials and keys can both be stored/managed locally by the end-user, or by using a remote self-hosted service, or a remote third party service.

ToDo: define relationship between Client and the Wallet.

# Use Cases

This is a non-exhaustive list of sample use cases.

## Issuance during Presentation - Wallet Initiated {#use-case-1}

A user comes across a verifier app that is requesting the user to present a credential, e.g., a driving license. The Wallet determines the requested credential type(s) from the credential presentation request and notifies the user that there is currently no matching credential in the Wallet. The Wallet determines an issuer capable of issuing a lacking credential and upon user consent sends the user to the issuer's user experience (web site or app). Upon being authenticated and providing consent to issue the credential into her Wallet, the user is sent back to the Wallet. The Wallet informs the user credential was successfully issued into the Wallet and is ready to be presented to the verifier app that originally requested presentation of that credential.

## Issuance during Presentation - Wallet Initiated (requires presentation of additional credentials during issuance) {#use-case-2}

A user comes across a verifier app that is requesting the user to present a credential, e.g., a university diploma. The Wallet determines the requested credential type(s) from the credential presentation request and notifies the user that there is currently no matching credential in the Wallet. The Wallet than offers the user a list of issuers, which might be based on an issuer list curated by the Wallet provider. The user picks the university she graduated from and is sent to that university's user experience (web site or app).  

The user logs in to the university, who determines that the respective user account is not verified yet. Among multiple identification options, the user chooses to present identity credential from her Wallet. The user is sent back to the Wallet where she consents to sharing requested credential(s) to the university. The user is sent back to the university user experience. Based on the presented credential, the university complete user verification, looks up user data in its database and offers to issue a diploma as a verifiable credential. 

Upon providing consent, the user is sent back to the Wallet. The Wallet informs the user credential was successfully issued into the Wallet and is ready to be presented to the verifier app that originally requested presentation of that credential.

ToDo: Suggest we remove this example - I am worried it is overcomplicated and might scare people away..

## Same Device Issuance - Issuer Initiated {#use-case-3}

While browing university's home page, the user finds a link "request your digital diploma". User  clicks on this link and is being redirected to a digital Wallet application. The Wallet notifies the user that an issuer offered to issue a diploma credential. User confirms this inquiry and is taken to the university's credential issuance service (using WebView in the Wallet app, browser app, etc.). After authenticating at the university and consenting to the issuance of a digital diploma, the user is sent back to the Wallet, where she can check the successful creation of the digital diploma.

## Cross Device Issuance - Issuer Initiated (with information pre-submitted by the User) {#use-case-4}

The user is starting a job at a new employer. An employer has requested the user to upload certain documents to the employee portal. Few days later, the user receives an email from the employer notifying her that the employee credential is ready and asking her to scan a QR code to retrieve it. User scans the QR code with her smartphone, which opens her Wallet. Meanwhile, the user has received a text message with a PIN code to her smartphone. After entering that PIN code in the Wallet for security reasons, the user confirms the credential issuance, and receives credential into the Wallet.

## Deferred Credential Issuance - Wallet Initiated {#use-case-5}

The user wants to obtain a digital criminal record. User chooses in her Wallet issuance of a digital criminal record and is sent to the issuance service of the responsible government authority. After logging in with eID and requesting the issuance of the digital record, the user is notified that the issuance will take few days due to necessary background checks by the authority. She confirms and is sent back to the Wallet. 

In the Wallet, the user sees indication that issuance of the digital record is under way. A few days later, the user receives a notification from her Wallet app that requested credential was successfully issued. When the user opens the Wallet, she is asked whether she wants to download the certificate. She confirms and the new credential is retrieved and stored in the Wallet.

Note that the issuance can have multiple characteristics, which can be combined depending on the use-cases: Authorization Code Flow or Pre-Authorized Code Flow; Wallet initiated or Issuer initiated; Same-device or Cross-device; and Just-in-time or Deferred. The concepts will be described in the following sections.

# Overview

This specification defines the following mechanisms to allow Wallet applications (acting as OAuth 2.0 and Credential Endpoint Clients) used by the End-User to request credential issuers (acting as OAuth 2.0 Authorization Servers and Credential Endpoint providers) to issue Verifiable Credentials via the Credential Endpoint:

* A newly defined Credential Endpoint from which credentials can be issued. See (#credential_endpoint).
* An optional mechanism for the Issuer to initiate the issuance. See (#issuance_initiation_endpoint).
* An extended Authorization Request syntax that allows to request issuance of a specific credential type. See (#credential-authz-request).
* An optional ability to bind an issued credential to a proof submitted by the Wallet in the Credential Request. Multiple proof types are supported. See (#credential_request). 
* A mechanism for the Deferred Credential Issuance. See (#deferred-credential-issuance).
* A mechanism for the Issuer to publish metadata about the credential it is capable of issuing. See (#server-metadata)
* A mechanism that allows issuance of multiple credentials of same or different type. See (#token-response) and (#credential-response).

The Wallet can request only one credential per Credential Request. However, using the same Access Token the Wallet can repeat Credential Requests to request issuance of 
  - multiple credentials of different types bound to the same proof, or
  - as well as multiple credentials of the same type bound to different proofs

The Wallet can also request presentation of credentials as means to authenticate the User during the Issuance Flow as illustrated in (#use-case-2).

## Authorized Code Flow Overview

Below is a diagram of a credential issuance using Authorization Code flow, when the user authentication happens at the Authorization Endpoint. 

The diagram is based on a Wallet initiated flow illustrated in (#use-case-1) and does not illustrate all of the optional features. 

ToDo: discuss if need to illustrate the verifier... per use-case-1

!---
~~~ ascii-art
+--------------+   +-----------+                                         +-------------+
| User         |   |   Wallet  |                                         |   Issuer    |
+--------------+   +-----------+                                         +-------------+  
        |    interacts   |                                                      |
        |--------------->|                                                      |
        |                | -----                                                |
        |                |      |  Obtains Issuer's server metadata             |
        |                | <----                                                |
        |                |                                                      |
        |                |  (1) Credential Authorization Request                |
        |                |      (type(s) of credentials to be issued)           |
        |                |----------------------------------------------------->|
        |                |                                                      |
        |   User Authentication / Consent                                       |
        |                |                                                      |
        |                |      Credential Authorization Response (code)        |
        |                |<-----------------------------------------------------|
        |                |                                                      |
        |                |  (3) Token Request (code)                            |
        |                |----------------------------------------------------->| 
        |                |      Token Response (access_token)                   |
        |                |<-----------------------------------------------------|    
        |                |                                                      |
        |                |  (3) Credential Request (access_token, proof(s))     |
        |                |----------------------------------------------------->| 
        |                |      Credential Response                             |
        |                |      (credential(s) OR acceptance_token)             |
        |                |<-----------------------------------------------------|             
~~~
!---
Figure: Issuance using Authorization code flow 

(1) Wallet sends an Credential Authorization Request to the Issuer's Authorization Endpoint. Issuer returns Authorization Response with the authorization code upon successfully authenticating and obtaining consent from the End-User. This step happens via a frontchannel, by redirecting the End-User via the user agents. This step is defined in (#authorization_endpoint).

(2) Wallet sends a Token Request to the Issuer's Token Endpoint with the authorization code obtained in step (2). Issuer returns an Access Token in the Token Request upon successfully validating authorization code. This step happens backchannel using server to server communication. This step is defined in (#token_endpoint).

(3) Wallet sends a Credential Request to the Issuer's Credential Endpoint with the Access Token and proof of control over the public key to which the the issued VC shall be bound. Upon successfully validating Access Token and a proof, the Issuer returns a VC in the Credential Response if it is able to issue a credential right away. This step is defined in (#credential_endpoint).

If the Issuer requires more time to issue a credential, the Issuer may returns an Acceptance Token to the Wallet with the information when the Wallet can start sending Deferred Credential Request to obtain an issued credential as defined in (#deferred-credential-issuance).

Note that this flow is based on OAuth and can be used not only with the code grant type as in the description, but with other grant types as well, other than implicit grant. 


# Pre-Authorized Code Flow Overview

Below is a diagram of a credential issuance using Pre-Authorized Code flow defined in (#pre-authorized-code-flow). It is a flow when the user authentication happens out of band, prior to the issuance flow, without utilizing the Authorization Endpoint. 

The diagram is based on an Issuer initiated flow illustrated in (#use-case-4) and does not illustrate all of the optional features.

!---
~~~ ascii-art
+--------------+   +-----------+                                         +-------------+
| User         |   |   Wallet  |                                         |   Issuer    |
+--------------+   +-----------+                                         +-------------+
        |                |  (0) User provides  information required             |  
        |                |      for the issuance of a certain credential        |
        |---------------------------------------------------------------------->|
        |                |                                                      |
        |                |  (1) Issuance Initiation Request                     |
        |                |      (pre-authorized_code)                           |
        |                |<-----------------------------------------------------|        
        |    interacts   |                                                      |
        |--------------->|                                                      |
        |                | -----                                                |
        |                |      |  Obtains Issuer's server metadata             |
        |                | <----                                                |
        |                |                                                      |
        |                |  (2) Token Request (pre_authorized_code)             |
        |                |----------------------------------------------------->| 
        |                |      Token Response (access_token)                   |
        |                |<-----------------------------------------------------|    
        |                |                                                      |
        |                |  (3) Credential Request (access_token, proof(s))     |
        |                |----------------------------------------------------->| 
        |                |      Credential Response                             |
        |                |      (credential(s))                                 |
        |                |<-----------------------------------------------------|             
~~~
!---
Figure: Issuance using Pre-Authorized code flow 

(0) The Issuer successfully obtains consent and user data required for the issuance of a requested credential from the user using Issuer specific business process, without using Authorization Endpoint.

(1) The flow begins as the Issuer generates an Issuance Initiation Request and communicates it to the Wallet, for example as a QR Code or as a deeplink. Wallet uses information in the Issuance Initiation Request to obtain issuer's metadata including details about the credential that this issuer wants to issue. This step is defined in (#issuance_initiation_endpoint).

(2) This step is the same as Step 3 in the Authorization Code Flow, but instead of authorization code, pre-authorized_code is sent in the Token Request. This step is defined in (#token_endpoint).

(3) This step is the same as Step 4 in the Authorization Code Flow. This step is defined in (#credential_endpoint).

It is important to note that anyone who possesses a valid pre-authorization_code would be able to receive a VC from the Issuer. Implementers MUST implement mitigations most suitable to the use-case. For more details and concrete mitigations, see (#security-considerations).

# Pre-Authorized Code Flow {#pre-authorized-code-flow} //WIP

This section specifies an additional flow to obtain an access token for credential issuance. It is intended to support scenarios where the user starts a process on an issuer's website that ultimately results in one or more credentials being issued to the user's Wallet. The process on the issuer's website may include uploading documents and presenting verifiable credentials to the issuer. Moreover, the End-user may be accessing the issuer's website on a device different from the one with the Wallet application so the credential issuance process needs to be transfered to another device where the user's Wallet resides. 

In contrast to the flow specified in (#endpoints), this flows is initiated by the Issuer when the credentials are "ready" and need to be "picked up" by the Wallet application. How the user provides information required for the issuance of a requested credential to the Issuer and the business processes conducted by the Issuer to issue a credential are out of scope of this specification.

1. To initiate the flow, the Issuer needs to communicates to the Wallet two parameters: the pre-authorized code and the credential type the code is valid for. The Issuer can send them in two different ways. Either the issuer sends them in an issuance initiation request to the Issuance Initiation Endpoint of the Wallet as described in (#issuance_initiation_request). Or the issuer renders a QR code containing the issuance initiation request with these parameters. In this case, the user scans the QR code with her Wallet on a different device in order to proceed. 
1. When initiating the flow, the Issuer also indicates if the PIN is required. If it is, sends PIN to the Wallet using a channel different from an issuance initiation request.
1. The Wallet sends the pre-authorized code to the issuer's Token Endpoint. This request MUST contain a user PIN if requested by the issuer. 
1. The issuer responds with an access token valid for credential issuance. 
1. The Wallet sends a credential issuance request to the credential endpoint as defined in (#credential_request) using the credential type as indicated in the first step from the issuer to the Wallet.
1. The issuer returns the requested credential as defined in (#credential-response). 

Steps 1 through 3 constitute a new kind of pre-authorized code flow that is implemented using an additional issuance initiation endpoint and a new OAuth grant type "urn:ietf:params:oauth:grant-type:pre-authorized_code". Steps 4 through 6 conform to the process specified in (#endpoints).

Note that the pre-authorized code is sent to the Token Endpoint and not to the Authorization Endpoint of the Issuer.

# Endpoints {#endpoints}

## Overview

This specification defines new endpoints as well as additional parameters to existing OAuth 2.0 endpoints required to implement the protocol outlined in the previous section. It also introduces a new authorization details type according to [@!I-D.ietf-oauth-rar] to convey the details about the credentials the Wallet wants to obtain. Aspects not defined in this specification are expected to follow [@!RFC6749]. it is RECOMMENDED to use PKCE as defined in [@!RFC7636] to prevent authorization code interception attacks.

ToDo: introduce RAR before this authorization endpoint description.

There are the following new endpoints: 

* Issuance Initiation Endpoint: An endpoint exposed by the Wallet that allows an issuer to initiate the issuance flow
* Credential Endpoint: this is an OAuth 2.0-protected API used to issue verifiable credentials
* Deferred Credential Endpoint: this endpoint is used for deferred issuance of verifiable credentials 

The following endpoints are extended:

* Client Metadata: new metadata parameter is added to allow a Wallet (acting as OAuth 2.0 client) to publish its issuance initiation endpoint.
* Server Metadata: New metadata parameters are added to allow the client to determine what types of verifiable credentials a particular OAuth 2.0 Authorization Server is able to issue along with additional information about formats and prerequisites.
* Authorization Endpoint: The `authorization_details` parameter is extended to allow clients to specify types of the credentials when requesting authorization for issuance. These extension can also be used via the Pushed Authorization Endpoint, which is recommended by this specification. 
* Token Endpoint: optional parameters are added to the token endpoint to provide the client with a nonce to be used for proof of possession of key material in a subsequent request to the credential endpoint. 

## Issuance Initiation Endpoint {#issuance_initiation_endpoint}

This endpoint is used by an issuer in case it is already in an interaction with a user that wishes to initate a credential issuance. It is used to pass available 
information relevant for the credential issuance to ensure a convenient and secure process. 

### Issuance Initiation Request {#issuance_initiation_request}

The issuer (or any other party wishing to kickstart an issuance into a Wallet) sends the request as a HTTP GET request or a HTTP redirect to the Issuance Initiation Endpoint URL.

The following request parameters are defined: 

* `issuer`: REQUIRED. The issuer URL of the credential issuer, the Wallet is requested to obtain one or more credentials from. 
* `credential_type`: REQUIRED. A JSON string denoting the type of the credential the Wallet shall request.
* `op_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the Wallet that is used to bind the sub-sequent authentication request with the Credential Issuer to a context set up during previous steps. If the client receives a value for this parameter, it MUST include it in the subsequent Authentication Request to the Credential Issuer as the `op_state` parameter value.
* `pre-authorized_code`: CONDITIONAL. The code representing the issuer's authorization for the Wallet to obtain credentials of a certain type. This code MUST be short lived and single-use. MUST be present in a pre-authorized code flow.
* `user_pin_required`: OPTIONAL. Boolean value specifying whether the issuer expects presentation of a user PIN along with the Token Request in a pre-authorized code flow. Default is `false`. This PIN is intended to bind the pre-authorized code to a certain transaction in order to prevent replay of this code by an attacker that, for example, scanned the QR code while standing behind the legit user. It is RECOMMENDED to send a PIN via a separate channel.

Below is a non-normative example of an issuance initiation request in an authorization code flow:

```
  GET /initiate_issuance?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &op_state=eyJhbGciOiJSU0Et...FYUaBy
```

The issuer MAY also render a QR code containing the request data in order to allow the user to scan the request using her Wallet app. 

The Wallet MUST consider the parameter values in the initiation request as not trustworthy since the origin is not authenticated and the message 
integrity is not protected. The Wallet MUST apply the same checks on the issuer that it would apply when the flow is started from the Wallet itself since the issuer is not trustworthy just because it sent the initiation request. An attacker might attempt to use an initation request to conduct a phishing or injection attack. 

The Wallet MUST NOT accept credentials just because this mechanism was used. All protocol steps defined in this draft MUST be performed in the same way as if
the Wallet would have started the flow. 

The Wallet MUST be able to process multiple occurences of the URL query parameters `credential_type` and/or `manifest_id`. Multiple occurences MUST be 
treated as multiple values of the respective parameter.

The Issuer MUST ensure the release of any privacy-sensitive data is legally based (e.g., if passing an e-mail address in the `login_hint` parameter).

Below is a non-normative example of an issuance initiation request in a pre-authorized code flow:

```
  GET /initiate_issuance?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
```

Below is a non-normative example of the issuance initiation Request displayed by the Issuer as a QR code:

```
openid_initiate_issuance://?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
    &user_pin_required=true
```

### Issuance Initiation Response

The Wallet is not supposed to create a response. UX control stays with the Wallet after completion of the process. 

## Credential Authorization Endpoint {#authorization_endpoint}

The Authorization Endpoint is used in the same manner as defined in [@!RFC6749] taking into account the recommendations given in [@!I-D.ietf-oauth-security-topics] and utilizes [@!I-D.ietf-oauth-rar].

In addition to the required basic Authorization Request, this section also defines how pushed Authorization Requests can be used to protect the Authorization Request payload and when the requests become large.

### `authorization_details` Request Parameter {#authorization-details}

Request parameter `authorization_type` defined in Section 2 of [@!I-D.ietf-oauth-rar] MUST be used to convey the details about the credentials the Wallet wants to obtain. This specification introduces a new authorization details type `openid_credential` and defines the following elements to be used with this authorization details type:

* `type` REQUIRED. JSON string that determines the authorization details type. MUST be set to `openid_credential` for the purpose of this specification.
* `credential_type`: REQUIRED. JSON string denoting the type of the requested credential.
* `format`: OPTIONAL. JSON string representing a format in which the credential is requested to be issued. Valid values are defined in the table in Section 6.7.3. and include `jwt_vp` and `ldp_vp`. Formats identifiers not in the table, MAY be defined by the profiles of this specification.
* `locations`: OPTIONAL. An array of strings that allows a client to specify the location of the resource server(s) allowing the AS to mint audience restricted access tokens. This data field is predefined in Section 2.2 of ([@!I-D.ietf-oauth-rar]).

Note: `credential_type` and `format` are used when the Client has not pre-obtained a Credential Manifest. `manifest_id` is used when the Client has pre-obtained a Credential Manifest. These two approaches MAY be combined in one request in different authorization details objects.

Note: The `credential_application` element defined in [@DIF.CredentialManifest] is not required by this specification.

Note: Passing the `format` to the Authorization Request is informational and allows the credential issuer to refuse early in case it does not support the requested format/credential combination. The client MAY request issuance of credentials in other formats as well later in the process at the credential endpoint.

[TBD: `locations` could enable a single authorization server to authorize access to different credential endpoints. Might be an architectural option we want to pursue.]

A non-normative example of an `authorization_details` object. 

```json=
{
   "type":"openid_credential",
   "credential_type":"https://did.example.org/healthCard",
   "format":"ldp_vc"
}
```
Note: applications MAY combine `openid_credential` with any other authorization details type in an Authorization Request.
 
### Credential Authorization Request {#credential-authz-request}

A credential Authorization Request is an OAuth 2.0 Authorization Request as defined in section 4.1.1 of [@!RFC6749], which requests to grant access to the credential endpoint as defined in (#credential-endpoint). It also follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

There are two possible ways to make a credential Authorization Request. One way is to use of the `authorization_details` request parameter as defined in (#authorization-details) with one or more authorization details objects of type `openid_credential`. The other is through the use of scopes as defined in (#credential-request-using-type-specific-scope).

A non-normative example of a credential Authorization Request using the `authorization_details` parameter  (uses PKCE as defined in [@!RFC7636]) (with line wraps within values for display purposes only).

```
HTTP/1.1 302 Found
Location: https://server.example.com/authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &authorization_details=%5B%7B%22type%22:%22openid_credential%22,%22credential_type
  %22:%22https://did.example.org/healthCard%22,%22format%22:%22ldp_vc%22%7D,%7B%22ty
  pe%22:%22openid_credential%22,%22credential_type%22:%22https://did.example.org/mDL
  %22%7D%5D
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

This particiular non-normative example requests authorization to issue two different credentials:

```json=
[
   {
      "type":"openid_credential",
      "credential_type":"https://did.example.org/healthCard",
      "format":"ldp_vc"
   },
   {
      "type":"openid_credential",
      "credential_type":"https://did.example.org/mDL"
   }
]
```

A non-normative example below shows a credential Authorization Request which is also an OAuth 2.0 Authorization Request (uses PKCE as defined in [@!RFC7636]) and includes a truncated `authorization_request` parameter.

```
HTTP/1.1 302 Found
Location: https://server.example.com/authorize?
  response_type=code
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &scope=openid%20email
  &authorization_details=%5B%7B%22type%22:%22openid_credential%...7D%5D
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

#### Credential Authorization Request using Type Specific Scope {#credential-request-using-type-specific-scope}

An alternative credential request syntax to that defined in (#credential-authz-request) involves using an OAuth 2.0 scope following the syntax defined below.

```
openid_credential:<credential-type>
```

The value of `<credential-type>` indicates the type of credential being requested, providers who do not understand the value of this scope in a request MUST ignore it entirely. The presence of a scope following this syntax in the request MUST be interpreted by the provider as a request for access to the credential endpoint as defined in (#credential-endpoint) for the specific credential type. Multiple scope parameters MAY be present in a single request whereby each occurrence MUST be interpreted individually.

Below is a non-normative example of a Credential Authorization Request scoped to a specific credential type (uses PKCE as defined in [@!RFC7636]).

```
HTTP/1.1 302 Found
Location: https://server.example.com/authorize?
  response_type=code
  &scope=openid_credential:healthCard
  &client_id=s6BhdRkqt3
  &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
  &code_challenge_method=S256
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

If a scope `openid_credential:<credential-type>` and the `authorization_details` request parameter containing objects of type `openid_credential` are both present in a single request, the provider MUST interpret these individually. However, if both request the same credential type, than the Issuer MUST follow the request as given by the authorization details object.

#### Additional Request Parameters

This specification defines the following additional request parameters that can be supplied in any credential Authorization Request:

* `Wallet_issuer`: OPTIONAL. JSON String containing the Wallet's OpenID Connect Issuer URL. The Issuer will use the discovery process as defined in [@!SIOPv2] to determine the Wallet's capabilities and endpoints. RECOMMENDED in Dynamic Credential Request.
* `user_hint`: OPTIONAL. JSON String containing an opaque user hint the Wallet MAY use in subsequent callbacks to optimize the user's experience. RECOMMENDED in Dynamic Credential Request.
* `op_state`: OPTIONAL. String value identifying a certain processing context at the credential issuer. A value for this parameter is typically passed in an issuance initation request from the issuer to the Wallet (see ((#issuance_initiation_request)). This request parameter is used to pass the `op_state` value back to the credential issuer. 

Note: When processing the Authorization Request, the issuer MUST take into account that the `op_state` is not guaranteed to originate from this issuer in all circumstances. It could have been injected by an attacker. 

#### Pushed Authorization Request

Use of Pushed Authorization Requests is RECOMMENDED to ensure confidentiality, integrity, and authenticity of the request data and to avoid issues due to large requests due to the query language or if message level encryption is used.

Below is a non-normative example of a Pushed Authorization Request  (uses PKCE as defined in [@!RFC7636]):

```
POST /op/par HTTP/1.1
    Host: as.example.com
    Content-Type: application/x-www-form-urlencoded

    &response_type=code
    &client_id=CLIENT1234
    &code_challenge=E9Melhoa2OwvFrEMTJguCHaoeK1t8URWbuGJSstw-cM
    &code_challenge_method=S256
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &authorization_details=...
```

Below is a non-normative example of a `authorization_details` parameter with `manifest_id`:

```json=
{
   "type":"openid_credential",
   "manifest_id":"WA-DL-CLASS-A",
   "format":"jwt_vc"
}
```

#### Dynamic Credential Request

This step is OPTIONAL. After receiving an Authorization Request from the Client, the Issuer MAY use this step to obtain additional credentials from the End-User. 

The Issuer MUST utilize [@OpenID4VP] and [@!SIOPv2] to dynamically request additional credentials. From a protocol perspective, the Issuer acts now as a verifier and sends a presentation request to the Wallet. The Client MUST have these credentials obtained prior to initiating a transaction with this Issuer. 

This provides the benefit of the Issuer being able to adhere to the principle of data minimization, for example by including only minimum requirements in the Credential Manifest knowing that it can supplement additional information if needed.

To enable dynamic callbacks of the issuer to the end-user's Wallet, the Wallet MAY provide additional parameters `Wallet_issuer` and `user_hint` defined in the Authorization Request section of this specification.

For non-normative examples of request and response, see section 11.6 in [@OpenID4VP].

Note to the editors: need to sort out credential issuer's client_id with Wallet and potentially add example with `Wallet_issuer` and `user_hint` 

### Successful Credential Authorization Response

Authorization Responses MUST be made as defined in [@!RFC6749].

Below is a non-normative example of a successful Authorization Response:

```
HTTP/1.1 302 Found
  Location: https://Wallet.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
```

### Credential Authorization Error Response

Credential Authorization Error Response MUST be made as defined in [@!RFC6749].

Below is a non-normative example of an unsuccessful credential authorization response.

```json=
HTTP/1.1 302 Found
Location: https://client.example.net/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value
```

## Token Endpoint {#token_endpoint}

The Token Endpoint issues an Access Token and, optionally, a Refresh Token and an ID Token in exchange for the authorization code that client obtained in a successful Credential Authorization Response. It is used in the same manner as defined in [@!RFC6749] and follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

### Token Request

Upon receiving a successful Authorization Response, a Token Request is made as defined in Section 4.1.3 of [@!RFC6749].

The following are the extension parameters to the Token Request used in a pre-authorized code flow:

* `pre-authorized_code`: REQUIRED. The code representing the authorization to obtain credentials of a certain type. 
* `user_pin`: OPTIONAL. String value containing a user PIN. This value MUST be present if `user_pin_required` was set to `true` in the Issuance Initiation Request. The string value MUST consist of maximum 8 numeric characters (the numbers 0 - 9).

Below is a non-normative example of a Token Request in an authorization code flow:

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


Below is a non-normative example of a Token Request in a pre-authorized code flow:

```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
  &pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
  &user_pin=493536
```

### Successful Token Response {#token-response}

Token Requests are made as defined in [@!RFC6749].

In addition to the response parameters defined in [@!RFC6749], the AS MAY return the following parameters:

* `c_nonce`: OPTIONAL. JSON string containing a nonce to be used to create a proof of possession of key material when requesting a credential (see (#credential_request)).
* `c_nonce_expires_in`: OPTIONAL. JSON integer denoting the lifetime in seconds of the `c_nonce`.
* `authorization_pending`: OPTIONAL. JSON Boolean. In pre-authorized code flow, the Token Request is still pending as the issuer is waiting for the end user interaction to complete. The client SHOULD repeat the Token Request. Before each new request, the client MUST wait at least the number of seconds specified by the "interval" response parameter. ToDo: clarify boolean.
* `interval`: OPTIONAL. The minimum amount of time in seconds that the client SHOULD wait between polling requests to the token endpoint in pre-authorized code flow.  If no value is provided, clients MUST use 5 as the default.

Upon receiving `pre-authorized_code`, the issuer MAY decide to interact with the end-user in the course of the Token Request processing, which might take some time. In such a case, the issuer SHOULD respond with the error `authorization_pending` and the new return parameter `interval`.

Note: Subject Identifier in the ID Token is the End-User's identifier.

Below is a non-normative example of a Token Response:

```
HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "token_type": "bearer",
    "expires_in": 86400,
    "c_nonce": "tZignsnFbp",
    "c_nonce_expires_in": 86400
  }
```

### Token Error Response

If the Token Request is invalid or unauthorized, the Authorization Server constructs the error response as defined as in Section 5.2 of OAuth 2.0 [@!RFC6749].

Below is a non-normative example Token Error Response:

```json=
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache
{
   "error": "invalid_request"
}
```

## Credential Endpoint {#credential_endpoint}

The Credential Endpoint issues a credential as approved by the End-User upon presentation of a valid Access Token representing this approval. 

Communication with the Credential Endpoint MUST utilize TLS. 

The client can request issuance of a credential of a certain type multiple times, e.g., to associate the credential with different DIDs/public keys or to refresh a certain credential.

If the access token is valid for requesting issuance of multiple credentials, it is at the client's discretion to decide the order in which to request issuance of multiple credentials requested in the Authorization Request.

### Binding the Issued Credential to the identifier of the End-User possessing that Credential {#credential-binding}

Issued credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the credential. Cryptographic binding allows the Verifier to verify during presentation that the End-User presenting a credential is the same End-User to whom it was issued. For non-cryptographic type of binding and credentials issued without any binding, see Implementations Considerations sections (#claim-based-binding) and (#no-binding). 

Note that claims in the credential are usually about the End-User who possesses it, but can be about another entity.

For cryptographic binding, the Client has the following options to provide cryptographic binding material for a requested credential as defined in (#credential_request):

1. Provide proof of control alongside key material (`proof` that includes `sub_jwk` or `did`)
1. Provide only proof of control without the key material (`proof` that does not include `sub_jwk` or `did`)

### Credential Request {#credential_request}

A Client makes a Credential Request by sending a HTTP POST request to the Credential Endpoint with the following parameters:

* `type`: REQUIRED. Type of a credential being requested. It corresponds to a `type` property in a Issuer metadata.
* `format`: OPTIONAL. Format of the credential to be issued. If not present, the issuer will determine the credential 
format based on the client's format default.
* `proof` OPTIONAL. JSON Object containing proof of possession of the key material the issued credential shall be 
bound to. The `proof` object MUST contain the following `proof_type` element which determines its structure:

  * `proof_type`: REQUIRED. JSON String denoting the proof type. 

This specification defines the following values for `proof_type`:

* `jwt`: objects of this type contain a single `jwt` element with a JWS [@!7515] as proof of possession. The JWT MUST contain the following elements:
    * `kid`: CONDITIONAL. JWT header containing the key ID. If the credential shall be bound to a DID, the `kid` refers to a DID URL which identifies a particular key in the DID Document that the credential shall be bound to. MUST NOT be present if `jwk` or `x5c` is present.
    * `jwk`: CONDITIONAL. JWT header containing the key material the new credential shall be bound to. MUST NOT be present if `kid` or `x5c` is present.
    * `x5c`: CONDITIONAL. JWT header containing a certificate or certificate chain corresponding to the key used to sign the JWT. This element may be used to convey a key attestation. In such a case, the actual key certificate will contain attributes related to the key properties. MUST NOT be present if `kid` or `jwk` is present.
    * `iss`: REQUIRED. MUST contain the client_id of the sender.
    * `aud`: REQUIRED. MUST contain the issuer URL of credential issuer.
    * `iat`: REQUIRED. MUST contain the instant when the proof was created.
    * `nonce`: REQUIRED. MUST contain a fresh nonce as provided by the issuer.

Note: if both `jwk` and `x5c` are present, the represented signing key MUST be the same in both. 

The `proof` element MUST incorporate a fresh nonce value generated by the credential issuer and the credential issuer's identifier (audience) to allow the credential issuer to detect replay. The way that data is incorporated depends on the proof type. In a JWT, for example, the nonce is conveyd in the `nonce` claims whereas the audience is conveyed in the `aud` claim. In a Linked Data proof, for example, the nonce is included as the `challenge` element in the proof object and the issuer (the intended audience) is included as the `domain` element.

The Issuer MUST validate that the `proof` is actually signed by a key identified in `kid` parameter.

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
  "typ": "JWT",
  "kid":"did:example:ebfeb1f712ebc6f1c276e12ec21/keys/1"
}.
{
  "iss": "s6BhdRkqt3",
  "aud": "https://server.example.com",
  "iat": "2018-09-14T21:19:10Z",
  "nonce": "tZignsnFbp"
}
```

Here is another example JWT not only proving possession of a private key but also providing key attestation data for that key:

```json
{
  "typ": "JWT",
  "alg": "ES256",
  "x5c":[<key certificate + certificate chain for attestation>]
}.
{
  "iss": "s6BhdRkqt3",
  "aud": "https://server.example.com",
  "iat": "2018-09-14T21:19:10Z",
  "nonce": "tZignsnFbp"
}
```

Below is a non-normative example of a Credential Request:

```
POST /credential HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: BEARER czZCaGRSa3F0MzpnWDFmQmF0M2JW

type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard
format=ldp%5Fvc
did=did%3Aexample%3Aebfeb1f712ebc6f1c276e12ec21
proof=%7B%22type%22:%22...-ace0-9c5210e16c32%22%7D
```

### Credential Response {#credential-response}

Credential Response can be Synchronous or Deferred. The Issuer may be able to immediately issue a requested credential and send it to the Client. In other cases, the Issuer may not be able to immediately issue a requested credential and would want to send a token to the Client to be used later to receive a credential when it is ready.

The following claims are used in the Credential Response:

* `format`: REQUIRED. JSON string denoting the credential's format
* `credential`: OPTIONAL. Contains issued credential. MUST be present when `acceptance_token` is not returned. MAY be a JSON string or a JSON object, depending on the credential format. See the table below for the format specific encoding requirements.
* `acceptance_token`: OPTIONAL. A JSON string containing a token subsequently used to obtain a credential. MUST be present when `credential` is not returned.
* `c_nonce`: OPTIONAL. JSON string containing a nonce to be used to create a proof of possession of key material when requesting a credential (see (#credential_request)).
* `c_nonce_expires_in`: OPTIONAL. JSON integer denoting the lifetime in seconds of the `c_nonce`.


The following table defines how issued credential MUST be returned in the `credential` claim in the Credential Response based on the credential format and the signature scheme. This specification does not require any additional encoding when credential format is already represented as a JSON object or a JSON string.

| Credential Signature Format | Credential Format Identifier | Signature Scheme | Need for encoding when returning in the Credential Response  |
|:------|:-----|:-----|:------------|
|JWS Compact Serialization | `jwt_vc`, `mdl_iso_json` | Credential conformant to the W3C Verifiable Credentials Data Model, or ISO/IEC 18013-5:2021 mobile driving licence (mDL) data model, and signed as a JWS Compact Serialization. | MUST be a JSON string. Credential is already a sequence of base64url-encoded values separated by period characters and MUST NOT be re-encoded. |
|JWS JSON Serialization | `jwt_vc`, `mdl_iso_json`| Credential conformant to the W3C Verifiable Credentials Data Model, or ISO/IEC 18013-5:2021 mobile driving licence (mDL) data model, and signed as a JWS JSON Serialization. | MUST be a JSON object. MUST NOT be re-encoded. |
|Data Integrity | `ldp_vc` | Credential conformant to the W3C Verifiable Credentials Data Model and signed with Data Integrity Proofs. | MUST be a JSON object. MUST NOT be re-encoded. |
| CL-Signatures |`ac_vc` | Credential conformant to the AnonCreds format as defined in the Hyperledger Indy project and signed using CL-signature scheme. | MUST be a JSON object. MUST NOT be re-encoded. |
| COSE |`mdl_iso_cbor`| Credential conformant to the ISO/IEC 18013-5:2021 mobile driving licence (mDL) data model, encoded as CBOR and signed as a COSE message. | MUST be a JSON string that is the base64url-encoded representation of the issued credential |

Credential formats expressed as binary formats MUST be base64url-encoded and returned as a JSON string.

Note that this table might be superceded by a registry in the future. Meanwhile, for interoperability, implementers MUST follow the requirements defined in the table above.

Below is a non-normative example of a Credential Response in a synchronous flow:

```
HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

{
  "format": "jwt_vc"
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
  Pragma: no-cache

{
  "acceptance_token": "8xLOxBtZp8",
  "c_nonce": "wlbQc6pCJp",
  "c_nonce_expires_in": 86400  
}
```

Note: Consider using CIBA Ping/Push OR SSE Poll/Push. Another option would be the Client providing `client_notification_token` to the Issuer, so that the issuer sends a Credential response upon successfully receiving a Credential request and then no need for the client to bring an acceptance token, the Issuer will send the credential once it is issued in a response that includes `client_notification_token`. (consider SSE options)

### Credential Issuer-Provided Nonce

Upon receiving a Credential Request, the credential issuer MAY require the client to send a proof of possession of the key material it wants a credential to be bound to. This proof MUST incorporate a nonce generated by the credential issuer. The credential issuer will provide the client with a nonce in an error response to any Credential Request not including such a proof or including an invalid proof. 

Below is a non-normative example of a Credential Response when the Issuer requires a `proof` upon receiving a Credential Request:

```
HTTP/1.1 400 Bad Request
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

{
  "error": "invalid_or_missing_proof"
  "error_description":
       "Credential issuer requires proof element in Credential Request"
  "c_nonce": "8YE9hCnyV2",
  "c_nonce_expires_in": 86400  
}
```

ToDo - 400 might not be a right answer.

## Deferred Credential Endpoint {#deferred-credential-issuance}

This endpoint is used to issue a credential previously requested at the credential endpoint in case the Issuer was not able to immediately issue this credential. 

### Deferred Credential Request

This is an HTTP POST request, which accepts an acceptance token as the only parameter. The acceptance token MUST be sent as access token in the HTTP header as shown in the following example.

```
POST /credential_deferred HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: BEARER 8xLOxBtZp8

```

### Deferred Credential Response

The deferred Credential Response uses the `format` and `credential` parameters as defined in (#credential-response). 

## Client Metadata 

This specification defines the following new Client Metadata parameter in addition to [@!RFC7591] for Wallets acting as OAuth client:

* `initiate_issuance_endpoint`: OPTIONAL. URL of the issuance initation endpoint of a Wallet. 

If the issuer is unable to perform discovery of the Issuance Initiation Endpoint URL, the following static URL is used: `openid_initiate_issuance:`.

## Server Metadata {#server-metadata}

This section extends the server metadata [@!RFC8414] to allow the RP to obtain information about the credentials an OP is capable of issuing.

This specification defines the following new Server Metadata parameters for this purpose:

* `credential_endpoint`: REQUIRED. URL of the OP's Credential Endpoint. This URL MUST use the `https` scheme and MAY contain port, path and query parameter components.

The following parameter MUST be used to communicates the specifics of the credential that the issuer supports issuance of:
* `credentials_supported`: REQUIRED. A JSON object containing a list of key value pairs, where the key is a string serving as an abstract identifier of the credential. This identifier is RECOMMENDED to be collision resistant - it can be globally unique, but does not have to be when naming conflicts are unlikely to arise in a given use case. The value is a JSON object. The JSON object MUST conform to the structure of the (#credential-metadata-object). 

* `credential_issuer`: OPTIONAL. A JSON object containing display properties for the credential issuer.
  * `display`: OPTIONAL. An array of objects, where each object contains display properties of a credential issuer for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
    * `name`: OPTIONAL. String value of a display name for the credential issuer.
    * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object with the same language identifier

### Credential Metadata Object {#credential-metadata-object}

This section defines the structure of the object that appears as the value to the keys inside the object defined for the `credentials_supported` metadata element.

* `display`: OPTIONAL. An array of objects, where each object contains display properties of a certain credential for a certain language. Below is a non-exhaustive list of parameters that MAY be included. Note that the display name of the credential is obtained from `display.name` and individual claim names from `claims.display.name` values.
  * `name`: REQUIRED. String value of a display name for the credential.
  * `locale`: OPTIONAL. String value that identifies language of this display object represented as language tag values defined in BCP47 [@!RFC5646]. Multiple `display` objects may be included for separate languages. There MUST be only one object with the same language identifier.
  * `logo`: OPTIONAL. A JSON object with information about the logo of the credential with a following non-exhaustive list of parameters that MAY be included:
    * `url`: OPTIONAL. URL where the Wallet can obtain a logo of the credential issuer.
    * `alt_text`: OPTIONAL. String value of an alternative text of a logo image.
  * `description`: OPTIONAL. String value of a description of the credential.
  * `background_color`: OPTIONAL. String value of a background color of the credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].
  * `text_color`: OPTIONAL. String value of a text color of the credential represented as numerical color values defined in CSS Color Module Level 37 [@!CSS-Color].

* `formats`: REQUIRED. A JSON object containing a list of key value pairs, where the key is a string identifying the format of the credential. Below is a non-exhaustive list of valid key values defined by this specification:
  * Claim Format Designations defined in [@!DIF.PresentationExchange], such as `jwt_vc` and `ldp_vc`
  * `mdl_iso`: defined in this specification to express a mobile driving licence (mDL) credential compliant to a data model and data sets defined in ISO/IEC 18013-5:2021 specification. 
  * `ac_vc`: defined in this specificaiton to express an AnonCreds credential format defined as part of the Hyperledger Indy project [Hyperledger.Indy].

The value in a key value pair is a JSON object detailing the specifics about the support for the credential format with a following non-exhaustive list of parameters that MAY be included:
  * `types`: REQUIRED. Array of strings representing a format specific type of a credential. This value corresponds to `type` in W3C [@!VC_DATA] and a `doctype` in ISO/IEC 18013-5 (mobile Driving License).
  * `cryptographic_binding_methods_supported`: OPTIONAL. Array of case sensitive strings that identify how the credential is bound to the identifier of the End-User who possesses the credential as defined in (#credential-binding). A non-exhaustive list of valid values defined by this specification are `did`, `mso`, and `none`.
  * `cryptographic_suites_supported`: OPTIONAL. Array of case sensitive strings that identify the cryptographic suites that are supported for the `cryptographic_binding_methods_supported`. Cryptosuites for credentials in `jwt_vc` format should use algorithm names defined in [IANA JOSE Algorithms Registry](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms). Cryptosuites for credentials in `ldp_vc` format should use signature suites names defined in [Linked Data Cryptographic Suite Registry](https://w3c-ccg.github.io/ld-cryptosuite-registry/).

* `claims`: REQUIRED. A JSON object containing a list of key value pairs, where the key identifies the claim offered in the credential. The value is a JSON object detailing the specifics about the support for the claim with a following non-exhaustive list of parameters that MAY be included:
  * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
  * `namespace`: OPTIONAL. String value of a namespace that the claim belongs to. Relevant for ISO/IEC 18013-5 (mobile Driving License) specification.
  * `value_type`: OPTIONAL. String value determining type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
  * `display`: OPTIONAL. An array of objects, where each object contains display properties of a certain claim in the credential for a certain language. Below is a non-exhaustive list of valid parameters that MAY be included:
    * `name`: OPTIONAL. String value of a display name for the claim.
    * `locale`: OPTIONAL. String value that identifies language of this object represented as language tag values defined in BCP47 [@!RFC5646]. There MUST be only one object with the same language identifier.

It is dependent on the credential format where the requested claims will appear.

The following example shows a non-normative example of the relevant entries in the OP metadata defined above

```
  HTTP/1.1 200 OK
  Content-Type: application/json

 {
  "credential_endpoint": "https://server.example.com/credential",
  "credentials_supported": {
    "university_degree" : {
      "display": [
        {
          "name": "University Credential",
          "locale": "en-US",
          "logo": {
            "url": "https://exampleuniversity.com/public/logo.png",
            "alternative_text": "a square logo of a university"
          },
          "background_color": "#12107c",
          "text_color": "#FFFFFF"
        },
        {
          "name": "在籍証明書",
          "locale": "jp-JA",
          "logo": {
            "url": "https://exampleuniversity.com/public/logo.png",
            "alternative_text": "大学のロゴ"
          },
          "background_color": "#12107c",
          "text_color": "#FFFFFF"
        }
      ],
      "formats": {
        "ldp_vc": {
          "types": [ "VerifiableCredential", "UniversityDegreeCredential" ],
          "binding_methods_supported": [ "did" ],
          "proof_types_supported": [ "Ed25519Signature2018" ]
        }
      },
      "claims": {
        "given_name": {
          "mandatory": false,
          "display": [
              {
              `name`: `Given Name`,
              `locale`: `en-US`
            },
            {
              `name`: `名前`,
              `locale`: `jp-JA`
            }
          ]  
        },
        "last_name": {},
        "degree": {},
        "gpa": {
          "mandatory": false,
          "value_type": "number",
            "display": [
              {
              `name`: `GPA`
              }
          ]
        }
      }
    },
    "credential_issuer": {
      "display": [
        {
          `name`: `Example University`,
          `locale`: `en-US`
        },
        {
          `name`: `サンプル大学`,
          `locale`: `jp-JA`
        }
      ]  
    }
  }
```

Note: The Client MAY use other mechanisms to obtain information about the verifiable credentials that an Issuer can issue.

# Security Considerations {#security-considerations}

## Trust between Wallet and Issuer

Credential Issuers often want to know what Wallet they are issuing credentials to and how private keys are managed for the following reasons:

* The issuer may wants to ensure that private keys are properly protected from exfiltration and replay in order to prevent an adversary from impersonating the legitimate credential holder by presenting her credential.
* The issuer may also want to ensure that the Wallet managing the credentials adheres to certain policies and, potentially, was audited and approved under a certain regulatory and/or commercial scheme. 

The following mechanisms in concert can be utilized to fulfill those objectives:

**Key attestation** is a mechanism where the device or security element in a device asserts the key management policy to the application creating and using this key. The Android Operating System, for example, provides apps with a certificate including a certificate chain asserting that a particular key is managed, for example, by an hardware security module [ref]. The Wallet can provide this data along with the proof of possession in the Credential Request (see (#credential_request) for an example) to allow the issuer to validate the key management policy. This indeed requires the issuer to rely on the trust anchor of the certificate chain and the respective key management policy. Another variant of this concept is the use of a Qualified Electronic Signature as defined by the eIDAS regulation [ref]. This signatures won't reveal the concrete properties of the associated private key to the issuer. However, due to the regulatory regime of eIDAS the issuer can deduce that the signing service manages the private keys according to this regime and fulfills very high security requirements. As another example, FIDO2 allows RPs to obtain an attestation along with the public key from a FIDO authenticator. That implicitly asserts the key management policy since the assertion is bound to a certain authenticator model and its key management capabilities. 

**App Attestation**: Key attestation, however, does not establish trust in the application storing the credential and producing presentation of that credential. App attestation as provided by mobile operating systems, e.g. iOS's DeviceCheck or Android's Safetynet, allows a server system to ensure it is communicating to a legitimate instance of its genuine app. Those mechanisms can be utilized to validate the internal integrity of the Wallet (as a whole).  

**Device Attestation**: Device Attestation attests the health of the device, on which the Wallet application is running, as a whole. It prevents compromises such as a malicious third party application tampering with a Wallet application managing keys and credentials, which cannot be captured only by obtaining app attestation of a Wallet application.

**Client authentication** allows a Wallet to authenticate with an issuer. In order to securely authenticate, the Wallet needs to utilize a backend component managing the key material and processing the secure communication with the credential issuer. The issuer may establish trust with the Wallet based on its own auditing or a trusted 3rd party attestation of the conformance of the Wallet to a certain policy.  

Directly using key, app and/or device attestations to proof certain capabilities towards an issuer is an obvious option. However, this at least requires platform mechanisms that issue signed assertions that 3rd parties can evaluate, which is not always the case (e.g. iOS's DeviceCheck). Also, such an approach creates dependencies on platform specific mechanisms, trust anchors, and platform specific identifiers (e.g. Android `apkPackageName`) and it reveals information about the internal design of a Wallet app. Implementers should take that consequences into account. 

The approach recommended by this specification is that the issuer relies on the OAuth 2.0 client authentication to establish trust in the Wallet and leaves it to the Wallet to ensure its internal integrity using app and key attestation (if required). This establishes a clean separation between the different hemispheres and a uniform interface irrespectively of the Wallet's architecture (e.g. native vs web Wallet). Client authentication can be performed with credentials registered with the issuer or with assertions issued to the Wallet by a 3rd party the issuer trusts for the purpose of client authentication.  

## Pre-authorized Code Flow

### Replay Prevention

The pre-authorized code flow is vulnerable to the replay of the pre-authorized code, because by design it is not bound to a certain device (as the authorization code flow does with PKCE). This means an attacker can replay at another device the pre-authorized code meant for a victime, e.g., the attacker can scan the QR code while it is displayed on the victim’s screen, and thereby getg access to the credential. Such replay attacks must be prevented using other means. The design facilitates the following options: 

* User PIN: the issuer might set up a PIN with the user (e.g. via text message or email), which needs to be presented in the Token Request.
* Callback to device where the transaction originated: upon receiving the Token Request, the issuer asks the user to confirm the originating device (device that displayed the QR code) that the issuer may proceed with the credential issuance process. While the issuer reaches out to the user on the other device to get confirmation, the issuer returns an `authorization_pending` error code to the Wallet (as described in (#pre-authz-token-response)). The Wallet is required to call the token endpoint again to obtain the access token. If the user does not confirm, the Token Request is returned with the `access_denied` error code. This flow gives the user on the originating device more control over the issuance process.

### PIN Code Phishing

An attacker might leverage the credential issuance process and the user's trust into the Wallet to phish PIN codes sent out by a different service that grant attacker access to services other than credential issuance. The attacker could setup a credential issuer site and in parallel to the issuance request trigger transmission of a PIN code to the user's phone from a service other than credential issuance, e.g. from a payment service. The user would then be asked to enter this PIN into the Wallet and since the Wallet sends this PIN to the token endpoint of the issuer (the attacker), the attacker would get access to the PIN code, and access to that other service. 

In order to cope with that issue, the Wallet is RECOMMENDED to interact with trusted issuers only. In that case, the Wallet would not process an Issuance Initiation Request with an untrusted issuer URL. The Wallet MAY also show to the user the endpoint or issuer it will be sending the PIN code and ask the user for confirmation.

## Credential Lifecycle Management 

The issuer is supposed to be responsible for the lifecycle of its credentials. This means the issuer will invalidate credentials if it deems appropriate, e.g. if it detects fraudulent behavior.

The Wallet is supposed to detect signs of fraudulant behavior related to the credential management in the Wallet (e.g. device rooting) and to act upon such signals. Options include credential revocation at the issuer and/or invalidation of the key material used for cryptograpic holder binding.  

# Implementation Considerations

## Claim-based Binding of the Credential to the End-User possessing the credential {#claim-based-binding}

Credential not cryptographically bound to the identifier of the End-User possessing it (see (#credential-binding)), should be bound to the End-User possessing the credential based on the claims included in that credential. 

In claim-based binding, no cryptographic binding material is provided. Instead, the issued credential includes user claims that can be used by the Verifier to verify possession of the credential by requesting presentation of existing forms of physical or digial identification that includes the same claims (e.g., a driver's license or other ID cards in person, or an online ID verification service).

## Binding of the Credential without Cryptographic Binding nor Claim-based Binding {#no-binding}

Some Issuers might choose issuing bearer credentials without either cryptographic binding nor claim-based binding, because they are meant to be presented without proof of possession.

One such use case is low assurance credentials such as coupons or tickets. 

Another use case is when the Issuer uses cryptographic schemes that can provide binding to the End-User possessing that credential without explicit cryptographic material being supplied by the application used by that End-User. For example, in the case of the BBS Signature Scheme, the issued credential itself is a secret and only derivation of a credential is presented to the Verifier. Effectively, credential is bound to the Issuer's signature on the credential, which becomes a shared secret transferred from the Issuer to the End-User.

# Privacy Considerations

TBD

{backmatter}

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

<reference anchor="DIF.CredentialManifest" target="https://identity.foundation/credential-manifest/">
        <front>
          <title>Presentation Exchange v1.0.0</title>
      <author fullname="Daniel Buchner">
            <organization>Microsoft</organization>
          </author>
          <author fullname="Brent Zunde">
            <organization>Evernym</organization>
          </author>
          <author fullname="Jace Hensley">
            <organization>Bloom</organization>
          </author>
          <author fullname="Daniel McGrogan">
            <organization>Workday</organization>
          </author>
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

# IANA Considerations

register "urn:ietf:params:oauth:grant-type:pre-authorized_code"

# Acknowledgements {#Acknowledgements}

We would like to thank David Chadwick, John Bradley, Mark Haine, Alen Horvat, Michael B. Jones, and David Waite for their valuable contributions to this specification.

# Notices

Copyright (c) 2022 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -06

   * added issuer metadata
   * made Credential Response more flexible regarding credential encoding 
   * changed file name to match specification name
   * renamed specification to reflect OAuth 2.0 being the base protocol

   -05

   * added support for pre-authorized code flow
   * changed base protocol to OAuth 2.0

   -04

   * added support for requesting credential authorization with scopes 
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
