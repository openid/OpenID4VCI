%%%
title = "OpenID Connect for Verifiable Credential Issuance"
abbrev = "openid-connect-4-vc-issuance"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-connect-4-verifiable-credential-issuance-1_0-04"
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

%%%

.# Abstract

This specification defines an extension of OpenID Connect to allow holders to request issuance of verifiable credentials in addition to the standard OpenID Connect assertions.

{mainmatter}

# Introduction

This specification extends OpenID Connect with support for issuance of verifiable credentials, e.g., in the form of W3C Verifiable Credentials. This allows existing OpenID Connect OPs to extend their service and become credential issuers. It also allows new applications built using Verifiable Credentials to utilize OpenID Connect as integration and interoperability layer between credential holders and issuers.

OpenID Connect is an obvious choice for this use case since it already allows Relying Parties to request identity assertions. Verifiable Credentials are very similar in that they allow an Issuer to assert End-User claims. In contrast to the identity assertions, a verifiable credential follows a pre-defined schema (the credential type) and is bound to key material allowing the holder to prove the legitimate possession of the credential. This allows direct presentation of the credential without involvement of the credential issuer. This specification caters for those differences.

# Terminology

Credential

A set of one or more claims made by an issuer (see [@VC_DATA]). Note that this definition differs from that in [@OpenID.Core].

Verifiable Credential (VC)

A verifiable credential is a tamper-evident credential that has authorship that can be cryptographically verified. Verifiable credentials can be used to build verifiable presentations, which can also be cryptographically verified (see [@VC_DATA]).

Presentation

Data derived from one or more verifiable credentials, issued by one or more issuers, that is shared with a specific verifier (see [@VC_DATA]).

Verified Presentation (VP)

A verifiable presentation is a tamper-evident presentation encoded in such a way that authorship of the data can be trusted after a process of cryptographic verification. Certain types of verifiable presentations might contain data that is synthesized from, but do not contain, the original verifiable credentials (for example, zero-knowledge proofs) (see [@VC_DATA]).

W3C Verifiable Credential Objects

Both verifiable credentials and verifiable presentations

Credential Manifests 

A resource format that defines preconditional requirements, Issuer style preferences, and other facets User Agents utilize to help articulate and select the inputs necessary for processing and issuance of a specified credential (see [@DIF.CredentialManifest]).

Deferred Credential Issuance

Issuance of credentials not directly in the response to a credential issuance request, but following a period of time that can be used to perform certain offline business processes.

Note: Map the Issuer terminology to the OpenID Connect's OP term

# Use Cases

## Holder-Initiated Credential Issuance

A user comes across an app where she needs to present a credential, e.g., a bank identity credential. She starts the presentation flow at this app and is sent to her wallet (e.g., via Self-Issued OpenID Provider v2 and OpenID Connect for Verifiable Presentations). The wallet determines the desired credential type(s) from the request and notifies the user that there is currently no matching credential in the wallet. The wallet now offers the user a list of suitable issuers, which might be based on an issuer list curated by the wallet publisher. The user picks one of those issuers and is sent to the issuer's user experience (web site or app). There the user authenticates and is asked for consent to issue the required credential into her wallet. She consents and is sent back to the wallet, where she is informed that a credential was successfully created and stored in the wallet.

## Holder-Initiated Credential Issuance (Credential Presentation as Prerequisites)

A user comes across an app where she needs to present a credential, e.g., a university diploma. She starts the presentation flow at this app and is sent to her wallet (e.g., via Self-Issued OpenID Provider v2 and OpenID Connect for Verifiable Presentations). The wallet determines the desired credential type(s) from the request and notifies the user that there is currently no matching credential in the wallet. The wallet now offers the user a list of suitable issuers, which might be based on an issuer list curated by the wallet publisher. The user picks one of those issuers (her university) and is notified that the issuer requires an identity credential as prerequisite for issuance of the diploma. The wallet also offers to send the existing bank identity credential to the issuer for that purpose. The user confirms and is sent to the issuer's user experience (web site or app). The issuer evaluates the bank identity credential, looks up the user in its database, finds her diploma and offers to issue a verifiable credential. The user consents and is sent back to the wallet, where she is informed that a diploma verifiable credential was successfully created and stored in the wallet.

## Holder-Initiated Credential Issuance (with On-Demand Credential Presentation)

A user comes across an app where she needs to present a credential, e.g., a university diploma. She starts the presentation flow at this app and is sent to her wallet (e.g., via Self-Issued OpenID Provider v2 and OpenID Connect for Verifiable Presentations). The wallet determines the desired credential type(s) from the request and notifies the user that there is currently no matching credential in the wallet. The wallet now offers the user a list of suitable issuers, which might be based on an issuer list curated by the wallet publisher. The user picks one of those issuers (her university). The user confirms and is sent to the issuer's user experience (web site or app). The user logs in to the university, which determines that the respective user account is not verified yet. The user is offered to either use a video chat for identification or to fetch a suitable identity credential from her wallet. The user decides to fetch the necessary credential from her wallet and is sent back. In the wallet, she picks a suitable credential and authorizes transfer to the university. The wallet sends her back to the university. Based on the bank identity credential, the university verifies her identity and looks up her data in its database. The university finds her diploma and offers to issue a verifiable credential. The user consents and is sent back to the wallet, where she is informed that a diploma verifiable credential was successfully created and stored in the wallet.

## Issuer-Initiated Credential Issuance

The user browses her university's home page, searching for a way to obtain a digital diploma. She finds the respective page, which shows a link "request your digital diploma". She clicks on this link and is being sent to her digital wallet. The wallet notifies her that an issuer offered to issue a diploma credential. She confirms this inquiry and is being sent to the university's credential issuance service. She logs in with her university login and is being asked to consent to the creation of a digital diploma. She confirms and is sent back to her wallet. There, she is notified of the successful creation of the digital diploma.

## Issuer-Initiated Credential Issuance (Cross-Device / Credential Retrieval Only)

The user visits the administration office of her university to obtain a digital diploma. The university staff checks her student card and looks up her university diploma in the university's IT system. The office staff then starts the issuance process. The user is asked to scan a QR Code to retrieve the digital diploma. She scans the code with her smartphone, which automatically starts her wallet, where she is notified of the offer to create a digital diploma (the verifiable credential). She consents, which causes the wallet to obtain and store the verifiable credential.

## Issuer-Initiated Credential Issuance (with Prerequisites)

The user navigates to her university's webpage to obtain a digital diploma where she is asked to scan a QR Code to start the retrieval process. She scans the code with her smartphone, which automatically starts her wallet, where she is notified of the prerequisite to present her digital student card already stored in the wallet to receive a digital diploma (the verifiable credential). She consents, which causes the wallet to obtain and store the verifiable credential.

## Deferred Credential Issuance

The user wants to obtain a digital criminal record certificate. She starts the journey in her wallet and is sent to the issuer service of the responsible government authority. She logs in with her eID and requests the issuance of the certificate. She is notified that the issuance of the certificate will take a couple of days due to necessary background checks by the authority. She confirms and is sent back to the wallet. The wallet shows a hint in the credential list indicating that issuance of the digital criminal record certificate is under way. A few days later, she receives a notification from her wallet app telling her that the certificate was successfully issued. She opens her wallet, where she is asked after startup whether she wants to download the certificate. She confirms and the new credential is retrieved and stored in her wallet.

# Requirements

This section describes the requirements this specification aims to fulfill beyond the use cases described above. 

* Proof of possession of key material
  * Support all kinds of proofs (e.g., signatures, blinded proofs) but also issuance w/o proof
  * Proofs must be protected against replay by using issuer provided nonce (see also https://datatracker.ietf.org/doc/html/draft-ietf-oauth-dpop#section-8    
  * The proof mechanisms shall be complementary to OAuth/OpenID Connect mechanisms for request signatures, client authentication, and PoP to allow for its parallel usage.
* It shall be possible to request a single credential as well to request multiple credentials in the same request. Examples of the latter include: 
  * credentials containing different claims for the same user (micro/mono credentials) bound to the same key material
  * batch issuance of multiple credentials of the same type bound to different key material (see mDL)
* It shall be possible to issue multiple credentials based on same consent (e.g., different formats and/or keys - `did:key` followed by `did:ebsi`)
* Support for deferred issuance of credentials
* User authentication and identification
  * Issuer shall be able to dynamically obtain further data and be able to authenticate the user at their discretion
  * Holder shall be able to pass existing credentials (as presentations) or identity assertions to the issuance flow
    * Assisted flow (utilizing credential manifest)
    * Presentations/assertions must be protected against replay
* It shall be possible to request standard OpenID Connect claims and credentials in the same flow (to implement wallet onboarding, see EBSI/ESSIF onboarding)
* Support for Credential metadata (holder shall be able to determine the types of credentials an issuer is able to issue)
* Ensure OP is authoritative for respective credential issuer (OP (OpenID Connect issuer URL) <-> Issuer ID (DID))
* Incorporate/utilize existing specs
  * W3C VC HTTP API(?)
  * DIF Credential manifest(?)

# Overview 

This specification defines following mechanisms to allow credential holders (acting as OpenID Connect Clients) to request credential issuers (acting as OpenID Connect OPs) to issue Verifiable Credentials via OpenID Connect:

* An optional mechanism to pre-obtain a Credential Manifest
* An extended authorization request syntax that allows to request credential types to be issued
* Replay prevention of the credentials optionally submitted by the End-User as an issuance input (`p_nonce`)
* Ability to bind an issued credential to a proof submitted by the Client
* A newly defined Credential Endpoint from which credentials can be issued one at a time
* A mechanism that allows issuance of multiple credentials of same or different type  (`c_nonce`)
* A mechanism for the Deferred Credential Issuance (`acceptance_token`)

The following figure shows the overall flow. 

!---
~~~ ascii-art
+--------------+   +-----------+                                         +-------------+
| User         |   |   Wallet  |                                         |   Issuer    |
+--------------+   +-----------+                                         +-------------+
        |                |             interact                                 |  
        |---------------------------------------------------------------------->|
        |                |          (0) initiate issuance                       |
        |                |<-----------------------------------------------------|        
        |    interacts   |                                                      |
        |--------------->|                                                      |
        |                |  (1) [opt] obtain credential manifest                |
        |                |----------------------------------------------------->|
        |                |              credential manifest                     |
        |                |<-----------------------------------------------------|
        |                |                                                      |
      (2) [opt] User selects credentials                                        |          
        |                |                                                      |
        |                |  (3) [opt] request presentation nonce                |
        |                |----------------------------------------------------->|
        |                |      presentation nonce                              |
        |                |<-----------------------------------------------------|
        |                |                                                      |
        |                |  (4) authorization req (claims, [opt] input, etc. )  |
        |                |----------------------------------------------------->|
        |                |                                                      |
        |     (4.1) User Login                                                  |
        |                |                                                      |
        |                |  (4.2) [opt] request additional VCs (OIDC4VP)        |
        |                |<-----------------------------------------------------| 
        |                |                                                      |
    (4.2.1) [opt] User selects credentials                                      |
        |                |                                                      |
        |                |  (4.2.2) VCs in Verifiable Presentations             |
        |                |----------------------------------------------------->| 
        |                |                                                      |
        |   (4.3) User consents to credential issuance                          |
        |                |                                                      |
        |                |  (5) authorization res (code)                        |
        |                |<-----------------------------------------------------|
        |                |                                                      |
        |                |  (6) token req (code)                                |
        |                |----------------------------------------------------->| 
        |                |      access_token, id_token                          |
        |                |<-----------------------------------------------------|    
        |                |                                                      |
        |                |  (7) credential req (access_token, proofs, ...)      |
        |                |----------------------------------------------------->| 
        |                |      credential req (credentials OR acceptance_token)|
        |                |<-----------------------------------------------------|   
        |                |                                                      |
        |                |  (8) [opt] poll_credentials (acceptance_token)       |
        |                |----------------------------------------------------->| 
        |                |      credentials OR not_ready_yet                    |
        |                |<-----------------------------------------------------|          
~~~
!---
Figure: Overall Credential Issuance Flow

This flow is based on OpenID Connect and the code grant type. Use with other grant types, such as CIBA, will be possible as well. 

The starting point is an interaction of the user with her wallet. The user might, for example,

* want to present a credential and found out there is no suitable credential present in her wallet or
* have visited the web site of a Credential Issuer and wants to obtain a credential from that issuer. 

(0) (OPTIONAL) The issuer sends a request to the wallet to initiate the issuance flow. This request contains information about the 
credential(s) the holder wants to obtain from that issuer, e.g., in the form of credential manifest IDs or credential types, and 
further data, e.g., hints about the user when the user is already logged in with the Issuer.

(1) (OPTIONAL) obtain credential manifest (as defined in [@DIF.CredentialManifest]) from the issuer with an information of which Verifiable Credentials the Issuer can issue, and optionally what kind of input from the user the Issuer requires to issue that credential.

Note: The wallet MAY also obtain the information about the credential issuer's capabilities using other means, which is out of scope of this specification. 

(2) (OPTIONAL) If the issuer expects certain credentials to be presented in the issuance flow, it requests the user to select and confirm presentation of those credentials.

(3) If the user has confirmed the presentation of certain credentials with the issuance request, the wallet prepares the process by obtaining a nonce from the issuer. This nonce will be used to prevent malicious wallets from being able to replay those presentations. 

(4) In this step, the wallet sends an authorization request to the issuer. This request determines
the types of verifiable credentials the wallet (on behalf of the user) wants to obtain. It MAY also
include verifiable presentations if required by the issuer. 

The wallet SHOULD use a pushed authorization request (see [@!RFC9126]) to first send the payload of 
the authorization request to the issuer and subsequently use the `request_uri` returned by the issuer in the authorization
request. This ensures integrity and confidentiality of the request data and prevents any issues raised by URL length restrictions 
regarding the authorization request URL.

Note: Signed and encrypted request objects would also ensure integrity and confidentiality. However, this approach would further
increase the URL size, which might decrease robustness of the process. 

The issuer takes over user interface control at this point and interacts with the user. The implementation of 
this step is at the discretion of the issuer.  

(4.1)  The issuer will typically authenticate the user in the first step of this process. For this purpose,
the issuer might

* use a local or federated login, potentially informed by an `id_token_hint` (see [@OpenID.Core]) or
* utilize, if present, verifiable presentations passed to the authorization request.

(4.2) (OPTIONAL) The issuers MAY call back to the wallet to fetch verifiable credentials it needs as
prerequisite to issuing the requested credentials. The decision of what credentials are requested may depend
on the user identity determined in step 4.1.

From a protocol perspective, the issuers acts now as verifier and sends a request as defined in OpenID Connect for Verifiable Presentations 
[@OIDC4VP] to the wallet. 

(4.2.1) (CONDITIONAL) The wallet shows the content of the presentation request to the user. The user selects the 
appropriate credentials and consents. 

(4.2.2) (CONDITIONAL) The wallet responds with one or more verifiable presentations to the issuer. 

(4.3) The issuer asks the user for consent to issue the requested credentials. 

(5) The issuer responds with an authorization code to the wallet. 

(6) The wallet exchanges the authorization code for an Access Token and an ID Token.

(7) This Access Token is used to request the issuance of the actual credentials. The types of credentials the 
wallet can request is limited to the types approved in the authorization request in (5). The credential request 
passes the key material the respective credential shall be bound to. If required by the issuer, the wallet also passes a 
proof of possession for the key material. This proof of possession uses the SHA-256 hash of the Access Token
as cryptographic nonce. This ensures replay protection of the proofs. The format of key material and proof of
possession depends upon the proof scheme and is expressed in a polymorphic manner at the protocol level.

The Issuer will either directly respond with the credentials or issue an Acceptance Token, which 
is used by the wallet to poll for completion of the issuance process. 

(8) (OPTIONAL) The wallet polls the issuer to obtain the credentials previously requested in 
step (6). The issuer either responds with the credentials or HTTP status code "202" indicating 
that the issuance is not completed yet. 

Note: If the issuer just wants to offer the user to retrieve a pre-existing credential, it can
encode the parameter set of step (6) in a suitable representation and allow the wallet to start 
with step (6). One option would be to encode the data into a QR Code.  

## Approaches to Present Input Credentials to an Issuer {#present_input_credentials}

This draft intentionally supports two different approaches for presenting credentials to the credential issuer, 
designated as "static" and "dynamic". 

* "static": the wallet determines in the initial steps of the process based on metadata obtained from the issuer,
which types of credentials are required to obtain credentials. If suitable credentials are available,
the wallet creates verifiable presentations, which are then sent along with the authentication/authorization 
request to the issuer. This approach is facilitated by the DIF Credential Manifest [@DIF.CredentialManifest]. It requires the 
wallet to obtain cryptographic nonces, which need to be included into the verifiable presentations
to prevent replay of those presentations. 
* "dynamic": the wallet only requests credentials with the authorization request. The issuer determines what 
credentials it might need as prerequisite to issue credentials while processing the authorization request. This
approach is equivalent to an OP reaching out to a third-party identity provider to log the user in or to obtain
further identity claims. The difference is that the claims source is dynamically determined since it is the 
particular wallet of the user. The dynamic approach allows the issuer to only request the credentials absolutely 
necessary since it can previously identify or even authenticate the user before requesting credentials. In best case,
the issuer already has all necessary data in place and does not need to fetch any credential.

The draft includes both approaches because there is no existing best practice. Given the lack of implementation
experience, it seems to be better to leave implementers the choice. If the community, based on
implementation experience, gravitates towards one or the other approach, the draft could
be simplified by removing one of the options.

# Endpoints

## Overview

This specification defines the new endpoints as well as additional parameters to existing OpenID Connect endpoints required to implement the protocol outlined in the previous section. Aspects not defined in this specification are expected to follow [OpenID.Core].

There are the following new endpoints: 

* Issuance Initiation Endpoint: An endpoint exposed by the wallet that allows an issuer to initiate the issuance flow
* Nonce Endpoint: this endpoint provides the RP with a nonce it will include in verifiable presentations sent to the authorization endpoint
* Credential Endpoint: this is the OAuth-protected API to issue verifiable credentials
* Deferred Credential Endpoint: this endpoint is used for deferred issuance of verifiable credentials 

The following endpoints are extended:

* Client Metadata: new metadata parameter is added to allow a wallet (acting as OpenID Connect RP) to publish its issuance initiation endpoint.  
* Server Metadata: New metadata parameters are added to allow the RP to determine what types of verifiable credentials a particular OP is able to issue along with additional information about formats and prerequisites.
* Authorization Endpoint: The `claims` parameter is extended to allow the RP to request authorization for issuance of one or more credentials. The authorization request is extended with parameters to convey verifiable presentations to the authorization process and further data to alternatively callback to the RP (acting as wallet) to request further verifiable credentials. These extensions can also be used via the Pushed Authorization Endpoint, which is recommended by this specification. 
* Token Endpoint: optional parameters are added to the token endpoint to provide the RP with a nonce to be used for proof of possession of key material in a subsequent request to the credential endpoint. 

## Client Metadata 

This specification defines the following new Client Metadata parameter for wallets acting as OpenID Connect RP:

* `initiate_issuance_endpoint`: OPTIONAL. URL of the issuance initation endpoint of a wallet. 

If the issuer is unable to perform discovery of the Issuance Initiation Endpoint URL, the following static URL is used: `openid_initiate_issuance:`.

## Server Metadata

The server metadata [@!OpenID.Discovery] is extended to allow the RP to obtain information about the verifiable credentials an OP supports. This extension uses [@DIF.CredentialManifest]. 

Credential Manifest contains information about which type of VCs the Issuer can issue, and, optionally, what kind of input the Issuer requires from the Client in the request to issue that credential. This allows for a static approach of presenting input credentials to the Issuer as defined in (#present_input_credentials).

This specification defines the following new Server Metadata parameter for this purpose:

* `credential_manifests`: OPTIONAL. A JSON array containing a list of Credential Manifests. This parameter enables Issuers to pass Credential Manifests in a single self-contained parameter.
* `credential_manifest_uris`: OPTIONAL. A JSON array containing a list of URIs referencing resources each containing a Credential Manifest. This parameter enables Issuers to list Credential Manifests by reference, rather than by value. The scheme used MUST be https.

The following example shows an OpenID Configuration containing an embedded credential manifest.

```
  HTTP/1.1 200 OK
  Content-Type: application/json

 {
   "issuer":"https://server.example.com",
   "authorization_endpoint":"https://server.example.com/connect/authorize",
   "token_endpoint":"https://server.example.com/connect/token",
   ...
   "credential_manifests":[
      {
         "id":"WA-DL-CLASS-A",
         "version":"0.1.0",
         "issuer":{
            "id":"did:example:123?linked-domains=3",
            "name":"Washington State Government"
          },
         "output_descriptors":[
            {
               "schema":"http://washington-state-schemas.org/1.0.0/driver-license.json",
               "id": "output descriptor 1"
            }
         ],
         "presentation_definition":{}
     }
   ]
  }
```

Note: The RP MAY use other mechanisms to obtain information about the verifiable credentials that an OP can issue.

## Issuance Initiation Endpoint

This endpoint is used by an issuer in case it is already in an interaction with a user that wishes to initate a credential issuance. It is used to pass available 
information relevant for the credential issuance to ensure a convenient and secure process. 

### Issuance Initiation Request {#issuance_initiation_request}

The issuer (or any other party wishing to kickstart an issuance into a certain wallet) sends the request as a HTTP GET request or a HTTP redirect to the Issuance Initiation Endpoint URL. 

The following request parameters are defined: 

* `issuer`: REQUIRED. The issuer URL of the credential issuer, the wallet is requested to obtain one or more credentials from. 
* `credential_type`: CONDITIONAL. A JSON string denoting the type of the credential the wallet shall request. MUST be present if `manifest_id` is not present.
* `manifest_id`: CONDITIONAL. A JSON String  refering to a credential manifests published by the credential issuer. MUST be present if `credential_type` is not present. 
* `login_hint`: OPTIONAL. Hint about the login identifier the End-User might use to log in at the Credential Issuer. If the client receives a value for this parameter, it MUST include it in the subsequent Authentication Request to the Credential Issuer as the `login_hint` parameter value.
* `op_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the wallet that is used to bind the sub-sequent authentication request with the Credential Issuer to a context set up during previous steps. If the client receives a value for this parameter, it MUST include it in the subsequent Authentication Request to the Credential Issuer as the `op_state` parameter value.  

The following is a non-normative example:

```
  GET /initiate_issuance?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &login_hint=max%40example%2Ecom
```

The issuer MAY also render a QR code containing the request data in order to allow the user to scan the request using her wallet app. 

The wallet MUST consider the parameter values in the initiation request as not trustworthy since the origin is not authenticated and the message 
integrity is not protected. The Wallet MUST apply the same checks on the issuer that it would apply when the flow is started from the Wallet itself since the issuer is not trustworthy just because it sent the initiation request. An attacker might attempt to use an initation request to conduct a phishing or injection attack. 

The wallet MUST NOT accept credentials just because this mechanism was used. All protocol steps defined in this draft MUST be performed in the same way as if
the wallet would have started the flow. 

The wallet MUST be able to process multiple occurences of the URL query parameters `credential_type` and/or `manifest_id`. Multiple occurences MUST be 
treated as multiple values of the respective parameter.

The AS MUST ensure the release of any privacy-sensitive data is legally based (e.g., if passing an e-mail address in the `login_hint` parameter).

### Issuance Initiation Response

The wallet is not supposed to create a response. UX control stays with the wallet after completion of the process. 

## Nonce Endpoint

The Nonce Endpoint provides the RP with a presentation nonce. 

The Client MUST obtain a presentation nonce from the Issuer, when the Client needs to submit certain pre-obtained credentials to the Issuer to meet the requirements in one of the Issuer's Credential Manifests. The Client MUST bind credentials it is submitting to the received presentation nonce. This step is necessary to prevent submitted VCs from being replayed by a malicious Client.

Communication with the Nonce Endpoint MUST utilize TLS. 

### Presentation Nonce Request

Clients MUST use the HTTP POST method to send the Presentation Nonce Request to the Nonce Server. The Request SHOULD NOT include any parameters.

The rules for client authentication as defined in [@!RFC6749] for token endpoint requests, including the applicable authentication methods, apply for the Nonce endpoint as well. If applicable, the `token_endpoint_auth_method` client metadata parameter [@!OpenID.Registration] indicates the registered authentication method for the client to use when making direct requests to the authorization server, including requests to the Nonce endpoint. Similarly, the `token_endpoint_auth_methods_supported` authorization server metadata [@!OpenID.Discovery] parameter lists client authentication methods supported by the authorization server when accepting direct requests from clients, including requests to the PAR endpoint.

Below is a non-normative example of a presentation nonce request using `client_secret_basic` Client Authentication:

```
  POST /nonce HTTP/1.1
    Host: server.example.com
    Content-Type: application/x-www-form-urlencoded
    Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
```

### Presentation Nonce Response

After receiving and validating a valid Presentation Nonce Request from the Client, the Nonce Server returns a successful response that includes a presentation nonce. The response uses the application/json media type.

The following parameter MUST be included in the response:

* `p_nonce`: REQUIRED. presentation nonce that the Client MUST include in the presentations when submitting input credentials in the Authorization Request.
* `expires_in`: OPTIONAL. The lifetime of the nonce in seconds.

Below is a non-normative example of a credential challenge response

```
 HTTP/1.1 200 OK
  Content-Type: application/json;charset=UTF-8
  Cache-Control: no-store
  Pragma: no-cache

  {
    "p_nonce": "fbe22300-57a6-4f08-ace0-9c5210e16c32",
    "expires_in": "3600"
  }
```

## Authorization Endpoint

The Authorization Endpoint is used in the same manner as defined in Section 3.1.2 of [@!OpenID.Core], with the exception of the differences specified in this section.

In addition to the required basic Authorization Request, this section also defines

* how pushed authorization requests can be used to protect the authorization request payload and when the requests become large, and
* an optional dynamic credential presentation request that may be used by the Issuer to dynamically request additional credentials after receiving an Authorization Request (see also (#present_input_credentials)).
 
### Credential Authorization Request {#credential-request}

A credential authorization request builds upon the OpenID Connect Authentication request defined in section 3.1.2.1 of OpenID Connect core, which request that the End-User be authenticated by the Authorization Server but also granted access to the credential endpoint as defined in (#credential-endpoint).

There are two possible ways to make a credential authorization request, one makes use of the claims request parameter as defined by section 5.5 of [@!OpenID.Core] with a new top level element called `credentials`. The other is through the use of scopes as defined in (#credential-request-using-type-specific-scope).

A non-normative example of a credential authorization request using the claims request object syntax.

```
HTTP/1.1 302 Found
Location: https://server.example.com/authorize?
  response_type=code
  &scope=openid
  &client_id=s6BhdRkqt3
  &state=af0ifjsldkj
  &claims=%7B%22credential%...%2dp_vc%22%7D%7D%5D%7D%7D
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

A credential authorization request that features the claims request parameter MUST decoded to contain the new top-level element of `credentials` defined by this specification, the value of this `credentials` element MUST be a JSON object that conforms to the following structure.

* `credentials`: JSON array containing one or more objects specifying credentials the Client is requesting to be issued. It MAY optionally contain references to verifiable presentations provided as prerequisite for credential issuance.

The following elements are used in each object in the `credentials` property:

* `type`: CONDITIONAL. A JSON string denoting the type of the requested credential. MUST be present if `manifest_id` is not present.
* `manifest_id`: CONDITIONAL. JSON String referring to a credential manifest published by the credential issuer. MUST be present if `type` is not present.
* `format`: OPTIONAL. A JSON string representing a format in which the credential is requested to be issued. Valid values defined by this specification are `jwt_vc` and `ldp_vc`. Profiles of this specification MAY define additional format values.

This specification defines the following additional parameters:

* `vp_token`: OPTIONAL. A parameter defined in [@OIDC4VP] used to convey required verifiable presentations. The verifiable presentations passed in this parameter MUST be bound to a `p_nonce` generated by the respective issuer from the Nonce Endpoint. 
* `presentation_submission`: OPTIONAL. JSON object as defined in [@DIF.CredentialManifest]. This object refers to verifiable presentations required for the respective credential according to the Credential Manifest and provided in an authorization request. All entries in the `descriptor_map` refer to verifiable presentations provided in the `vp_token` authorization request parameter.
* `wallet_issuer`: OPTIONAL. JSON String containing the wallet's OpenID Connect Issuer URL. The Issuer will use the discovery process as defined in [@SIOPv2] to determine the wallet's capabilities and endpoints. RECOMMENDED in Dynamic Credential Request.
* `user_hint`: OPTIONAL. JSON String containing an opaque user hint the wallet MAY use in sub-sequent callbacks to optimize the user's experience. RECOMMENDED in Dynamic Credential Request.
* `op_state`: OPTIONAL. String value identifying a certain processing context at the credential issuer. A value for this parameter is typically passed in an issuance initation request from the issuer to the wallet (see ((#issuance_initiation_request)). This request parameter is used to pass the `op_state` value back to the credential issuer. 

Note: When processing the authorization request, the issuer MUST take into account that the `op_state` is not guaranteed to originate from this issuer. It could have been injected by an attacker. 

Below is a non-normative example of an authorization request:
```
  GET /authorize?
    response_type=code
    &client_id=s6BhdRkqt3 
    &redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
    &scope=openid
    &claims=%7B%22credential%...%2dp_vc%22%7D%7D%5D%7D%7D
    &state=af0ifjsldkj
```

Below is a non-normative example of a `claims` parameter with `type` claims:

```json=
{
   "credentials":[
      {
         "type":"https://did.example.org/healthCard",
         "format":"ldp_vc"
      },
      {
         "type":"https://did.exmaple.org/mDL"
      }
   ]
}
```

Note: `type` and `format` are used when the Client has not pre-obtained a Credential Manifest. `manifest_id` and `presentation_submission` are used when the Client has pre-obtained a Credential Manifest. These two approaches MAY be combined in one request.

Note: Passing the `format` to the authorization request is informational and allows the credential issuer to refuse early in case it does not support the requested format/credential combination. The client MAY request issuance of credentials in other formats as well later in the process at the credential endpoint.

Note: The `credential_application` element defined in [@DIF.CredentialManifest] is not required by this specification.

#### Credential Authorization Request using Type Specific Scope {#credential-request-using-type-specific-scope}

An alternative credential request syntax to that defined in (#credential-request) involves using an OAuth2 scope following the syntax defined below.

```
openid_credential:<credential-type>
```

The value of `<credential-type>` indicates the type of credential being requested, providers who do not understand the value of this scope in a request MUST ignore it entirely. The presence of a scope following this syntax in the request MUST be interpreted by the provider as a request for access to the credential endpoint as defined in (#credential-endpoint) for the specific credential type. Multiple occurrences of this scope MAY be present in a single request whereby each occurrence MUST be interpreted individually.

A non-normative example of a credential request scoped to a specific credential type.

```
HTTP/1.1 302 Found
Location: https://server.example.com/authorize?
  response_type=code
  &scope=openid%20openid_credential:healthCard
  &client_id=s6BhdRkqt3
  &state=af0ifjsldkj
  &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
```

If both the `claims` request object with `credentials` top-level element and an instance of a `openid_credential:<credential-type>` scope are present in a single request the provider MUST interpret these individually. However, if both the scopes are requesting the same credential type then the provider MUST follow the request as given by the `claims` request object.

#### Obtaining Credentials required in Credential Manifest

This step is OPTIONAL. It is performed prior to the Authorization Request when the obtained Credential Manifest includes `presentation_definition` and requires the Client to present certain credentials in the authorization request in the `vp_token` parameter. The Client MUST obtain those credentials prior to initiating a transaction with this Issuer.

Performing issuance based on the credentials submitted by the Client provides the benefit of the Issuer being able to issue a credential without necessarily having information about that user being stored in its database.

#### Pushed Authorization Request

Use of Pushed Authorization Requests is RECOMMENDED to ensure confidentiality, integrity, and authenticity of the request data and to avoid issues due to large requests due to the query language or if message level encryption is used.

Below is a non-normative example of a Pushed Authorization Request:

```
POST /op/par HTTP/1.1
    Host: as.example.com
    Content-Type: application/x-www-form-urlencoded

    &response_type=code
    &client_id=CLIENT1234
    &nonce=duk681S8n00GsJpe7n9boxdzen
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &scope=openid
    &claims=...
    &vp_token=...
```

Below is a non-normative example of a `claims` parameter with `manifest_id` and `presentation_submission`:

```json=
{
   "credentials":[
      {
         "manifest_id":"WA-DL-CLASS-A",
         "format":"ldp_vc",
         "presentation_submission":{
            "id":"a30e3b91-fb77-4d22-95fa-871689c322e2",
            "definition_id":"32f54163-7166-48f1-93d8-ff217bdb0653",
            "descriptor_map":[
               {
                  "id":"input_1",
                  "format":"jwt_vc",
                  "path":"$.verifiableCredential[0]"
               }
            ]
         }
      }
   ]
}
```

#### Dynamic Credential Request

This step is OPTIONAL. After receiving an Authorization Request from the Client, the Issuer MAY use this step to obtain additional credentials from the End-User. The Issuer MUST utilize [@OIDC4VP] and [@SIOPv2] to dynamically request additional credentials.

This provides the benefit of the Issuer being able to adhere to the principle of data minimization, for example by including only minimum requirements in the Credential Manifest knowing that it can supplement additional information if needed.

To enable dynamic callbacks of the issuer to the end-user's wallet, the wallet MAY provide additional parameters `wallet_issuer` and `user_hint` defined in the Authorization Request section of this specification.

For non-normative examples of request and response, see section 11.6 in [@!OIDC4VP].

Note to the editors: need to sort out credential issuer's client_id with wallet and potentially add example with `wallet_issuer` and `user_hint` 

### Successful Authorization Response

Authentication Responses MUST be made as defined in Section 3.1.2.5 of [@!OpenID.Core].

Below is a non-normative example of
```
HTTP/1.1 302 Found
  Location: https://wallet.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj
```

### Authentication Error Response

Authentication Error Response MUST be made as defined in section 3.1.2.6 of [@!OpenID.Core].

The following is a non-normative example of an unsuccessful token response.

```json=
HTTP/1.1 302 Found
Location: https://client.example.net/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value
    &state=af0ifjsldkj
```

## Token Endpoint

The Token Endpoint issues an Access Token, an ID Token, and optionally a Refresh Token in exchange for the authorization code obtained in a successful Authorization Response. It is used in the same manner as defined in Section 3.1.3 of [@!OpenID.Core], with the exception of the differences specified in this section.

### Token Request

Upon receiving a successful Authentication Request, a Token Request is made as defined in Section 3.1.3.1 of [@!OpenID.Core].

Below is a non-normative example of a token request:
```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
  grant_type=authorization_code
  &code=SplxlOBeZQQYbYS6WxSbIA
  &redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
  
```

### Successful Token Response

Token Requests are made as defined in Section 3.1.3.3 of [@!OpenID.Core].

In addition to the response parameters defined in Section 3.1.3.3 of [@!OpenID.Core], the OP MAY return the following parameters:

* `c_nonce`: OPTIONAL. JSON string containing a nonce to be used to create a proof of possession of key material when requesting a credential (see (#credential_request)).
* `c_nonce_expires_in`: OPTIONAL. JSON integer denoting the lifetime in seconds of the `c_nonce`.

Note: Subject Identifier in the ID Token is the End-User's identifier.

Below is a non-normative example of a token response:
```
HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp..sHQ",
    "token_type": "bearer",
    "expires_in": 86400,
    "id_token": "eyJodHRwOi8vbWF0dHIvdGVuYW50L..3Mz",
    "c_nonce": "tZignsnFbp",
    "c_nonce_expires_in": 86400
  }
```

### Token Error Response

If the Token Request is invalid or unauthorized, the Authorization Server constructs the error response as defined as in Section 5.2 of OAuth 2.0 [RFC6749].

The following is a non-normative example Token Error Response:

```json=
HTTP/1.1 400 Bad Request
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache
{
   "error": "invalid_request"
}
```

## Credential Endpoint

The Credential Endpoint issues a credential as approved by the End-User upon presentation of a valid Access Token representing this approval. 

Communication with the Credential Endpoint MUST utilize TLS. 

The client can request issuance of a credential of a certain type multiple times, e.g., to associate the credential with different DIDs/public keys or to refresh a certain credential.

If the access token is valid for requesting issuance of multiple credentials, it is at the client's discretion to decide the order in which to request issuance of multiple credentials requested in the Authorization Request.

### Credential Request {#credential_request}

A Client makes a Credential Request by sending a HTTP POST request to the Credential Endpoint with the following parameters:

* `type`: REQUIRED. Type of credential being requested. It corresponds to a `schema` property in a Credential Manifest obtained in a setup phase.
* `format`: OPTIONAL. Format of the credential to be issued. If not present, the issuer will determine the credential 
format based on the client's format default.
* `sub_jwk`: OPTIONAL. The key material the new credential shall be bound to. MUST NOT be present if `did` is present.
* `did`: OPTIONAL. The DID the credential shall be bound to. `sub_jwk` and `did` are mutually exclusive. MUST NOT be present if `sub_jwk` is present.
* `proof` OPTIONAL. JSON Object containing proof of possession of the key material the issued credential shall be 
bound to. The Client MAY provide this claim in addition to a `did` claim. At the minimum, the following parameters MUST be included. The `proof` structure depends on the proof type and other parameters MAY be included.

  * `type`: REQUIRED. JSON String denoting the proof type.
  * `verificationMethod` REQUIRED. Cryptographically resolvable identifier referencing a public key to verify the End-User's control over the associated private key. The base URI of this identifier MUST match the DID given in `did` parameter. It MAY contain relative path components, query parameters, and fragment identifiers.
  * `jws` CONDITIONAL. A signature performed by a key that can be obtained by an identifier in verificationMethod.

The `proof` element MUST incorporate a fresh nonce value generated by the credential issuer and the credential issuer's identifier (audience) to allow the credential issuer to detect replay. The way that data is incorporated depends on the proof type. In a Linked Data proof, for example, the nonce is included as the `challenge` element in the proof object and the issuer (the intended audience) is included as the `domain` element.

The Client has three options to provide binding material for a requested credential:

1. provide `sub_jwk`
1. provide `did`
1. provide `proof` in addition to a `did`. When it is recommended to add `proof`, see Security Considerations section.

Below is a non-normative example of a `proof` parameter:

```json
{
  "type": "RsaSignature2018",
  "created": "2018-09-14T21:19:10Z",
  "proofPurpose": "authentication",
  "verificationMethod": "did:example:ebfeb1f712ebc6f1c276e12ec21/keys/1",
  "challenge": "2H4dB9xl-FZQL-pixV-WJk0eOt4CXQ-1NXKW",
  "domain": "https://issuer.example.com",
  "jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..l9d0YHjcFAH2H4dB9xlWFZQLUpixVCWJk0eOt4CXQe1NXKWZwmhmn9OQp6YxX0a2LffegtYESTCJEoGVXLqWAA",
}
```

Below is a non-normative example of a credential request:

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

### Credential Response {#credential_response}

Credential Response can be Synchronous or Deferred. The Issuer may be able to immediately issue a requested credential and send it to the Client. In other cases, the Issuer may not be able to immediately issue a requested credential and would want to send a token to the Client to be used later to receive a credential when it is ready.

The following claims are used in the Credential Response:

* `format`: REQUIRED. JSON string denoting the credential's format
* `credential`: OPTIONAL. JSON string that is the base64url encoded representation of the issued credential. MUST be present when `acceptance_token` is not returned. 
* `acceptance_token`: OPTIONAL. A JSON string containing a token subsequently used to obtain a credential. MUST be present when `credential` is not returned.
* `c_nonce`: OPTIONAL. JSON string containing a nonce to be used to create a proof of possession of key material when requesting a credential (see (#credential_request)).
* `c_nonce_expires_in`: OPTIONAL. JSON integer denoting the lifetime in seconds of the `c_nonce`.

Below is a non-normative example of a credential response in a synchronous flow:

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

Below is a non-normative example of a credential response in a deferred flow:

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

Note: Consider using CIBA Ping/Push callback. Another option would be the Client providing `client_notification_token` to the Issuer, so that the issuer sends a Credential response upon successfully receiving a Credential request and then no need for the client to bring an acceptance token, the Issuer will send the credential once it is issued in a response that includes `client_notification_token`.

### Credential Issuer-Provided Nonce

Upon receiving a Credential Request, the credential issuer MAY require the client to send a proof of possession of the key material it wants a credential to be bound to. This proof MUST incorporate a nonce generated by the credential issuer. The credential issuer will provide the client with a nonce in an error response to any credential request not including such a proof or including an invalid proof. 

Below is a non-normative example of a Credential Response when the Issuer requires a `proof` upon receiving a Credential Request:

```
HTTP/1.1 400 Bad Request
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

{
  "error": "invalid_or_missing_proof"
  "error_description":
       "Credential issuer requires proof element in credential request"
  "c_nonce": "8YE9hCnyV2",
  "c_nonce_expires_in": 86400  
}
```

## Deferred Credential Endpoint

This endpoint is used to issue a credential previously requested at the credential endpoint in case the Issuer was not able to immediately issue this credential. 

### Deferred Credential Request

This is a HTTP POST request, which accepts an acceptance token as only parameter. The acceptance token MUST be sent as access token in the HTTP header as shown in the following example.

```
POST /credential_deferred HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded
Authorization: BEARER 8xLOxBtZp8

```

### Deferred Credential Response

The deferred credential response uses the `format` and `credential` parameters as defined in (#credential_response). 

# Security Considerations

## Proving Control of a DID Presented as Binding Material

Some DID Methods do not require the End-User identified by a DID to also be a controller of a private key associated to a public key in a DID Document tied to that DID. In these cases, it is RECOMMENDED that in the Credential Request, the Client provides a signature using the private key tied to a DID in a `proof` claim, in addition to a `did` claim.

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

<reference anchor="OIDC4VP" target="https://openid.net/specs/openid-connect-4-verifiable-presentations-1_0.html">
      <front>
        <title>OpenID Connect for Verifiable Presentations</title>
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
       <date day="17" month="December" year="2021"/>
      </front>
</reference>

<reference anchor="OpenID.Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html">
  <front>
    <title>OpenID Connect Discovery 1.0 incorporating errata set 1</title>
    <author initials="N." surname="Sakimura" fullname="Nat Sakimura">
      <organization>NRI</organization>
    </author>
    <author initials="J." surname="Bradley" fullname="John Bradley">
      <organization>Ping Identity</organization>
    </author>
    <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros">
      <organization>Google</organization>
    </author>
    <author initials="E." surname="Jay" fullname="Edmund Jay">
      <organization> Illumila </organization>
    </author>
   <date day="8" month="Nov" year="2014"/>
  </front>
</reference>

<reference anchor="OpenID.Registration" target="https://openid.net/specs/openid-connect-registration-1_0.html">
        <front>
          <title>OpenID Connect Dynamic Client Registration 1.0 incorporating errata set 1</title>
          <author fullname="Nat Sakimura">
            <organization>NRI</organization>
          </author>
          <author fullname="John Bradley">
            <organization>Ping Identity</organization>
          </author>
          <author fullname="Michael B. Jones">
            <organization>Microsoft</organization>
          </author>
          <date day="8" month="Nov" year="2014"/>
        </front>
 </reference>

# IANA Considerations

TBD

# Acknowledgements {#Acknowledgements}

We would like to thank John Bradley, Alen Horvat, Michael B. Jones, and David Waite for their valuable contributions to this specification.

# Notices

Copyright (c) 2022 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -04

   * added support for requesting credential authorization with scopes 

   -03

   * added issuance initiation endpoint
   * Applied cleanups suggested by Mike Jones post adoption.

   -02

   * Adopted as WG document

   -01

   * Added Request & Response syntax and descriptions
   * Reworked and extended sequence diagram

   -00 

   *  initial revision
