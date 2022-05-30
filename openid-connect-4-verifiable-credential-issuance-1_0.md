%%%
title = "OpenID Connect for Verifiable Credential Issuance"
abbrev = "openid-connect-4-vc-issuance"
ipr = "none"
workgroup = "OpenID Connect"
keyword = ["security", "openid", "ssi"]

[seriesInfo]
name = "Internet-Draft"
value = "openid-connect-4-verifiable-credential-issuance-1_0-05"
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

This specification defines an API and corresponding OAuth-based authorization mechanisms for issuance of verifiable credentials. This specification can be combined with OpenID Connect to obtain identity assertions along with verifiable credentials. 

{mainmatter}

# Introduction

This specification defines an API designated as Credential Endpoint and corresponding OAuth-based authorization mechanisms for issuance of verifiable credentials, e.g., in the form of W3C Verifiable Credentials. This allows existing OAuth deployments and OpenID Connect OPs to extend their service and become credential issuers. It also allows new applications built using Verifiable Credentials to utilize OAuth and OpenID Connect as integration and interoperability layer.

OpenID Connect would be an obvious choice for this use case since it already allows Relying Parties to request identity assertions. However, early implementation experience suggests adoption is made easier if the base protocol only focuses on the credential issuance, without mandating certain [@OpenID.Core] features, such as asserting a stable `sub` values. Thus the working group decided to use OAuth as a base protocol. Deployments can nevertheless provide a combined implementation of OpenID Connect and the Credential Endpoint since OpenID Connect is built on top of OAuth. 

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

W3C Verifiable Credential Objects

Both verifiable credentials and verifiable presentations

Credential Manifests 

A resource format that defines preconditional requirements, Issuer style preferences, and other facets User Agents utilize to help articulate and select the inputs necessary for processing and issuance of a specified credential (see [@DIF.CredentialManifest]).

Deferred Credential Issuance

Issuance of credentials not directly in the response to a credential issuance request, but following a period of time that can be used to perform certain offline business processes.

# Use Cases

## End-User Initiated Credential Issuance

A user comes across an app where she needs to present a credential, e.g., a bank identity credential. She starts the presentation flow at this app and is sent to her wallet (e.g., via Self-Issued OpenID Provider v2 and OpenID Connect for Verifiable Presentations). The wallet determines the desired credential type(s) from the request and notifies the user that there is currently no matching credential in the wallet. The wallet now offers the user a list of suitable issuers, which might be based on an issuer list curated by the wallet publisher. The user picks one of those issuers and is sent to the issuer's user experience (web site or app). There the user authenticates and is asked for consent to issue the required credential into her wallet. She consents and is sent back to the wallet, where she is informed that a credential was successfully created and stored in the wallet.

## End-User Initiated Credential Issuance (with On-Demand Credential Presentation)

A user comes across an app where she needs to present a credential, e.g., a university diploma. She starts the presentation flow at this app and is sent to her wallet (e.g., via Self-Issued OpenID Provider v2 and OpenID Connect for Verifiable Presentations). The wallet determines the desired credential type(s) from the request and notifies the user that there is currently no matching credential in the wallet. The wallet now offers the user a list of suitable issuers, which might be based on an issuer list curated by the wallet publisher. The user picks one of those issuers (her university). The user confirms and is sent to the issuer's user experience (web site or app). The user logs in to the university, which determines that the respective user account is not verified yet. The user is offered to either use a video chat for identification or to fetch a suitable identity credential from her wallet. The user decides to fetch the necessary credential from her wallet and is sent back. In the wallet, she picks a suitable credential and authorizes transfer to the university. The wallet sends her back to the university. Based on the bank identity credential, the university verifies her identity and looks up her data in its database. The university finds her diploma and offers to issue a verifiable credential. The user consents and is sent back to the wallet, where she is informed that a diploma verifiable credential was successfully created and stored in the wallet.

## Issuer-Initiated Credential Issuance

The user browses her university's home page, searching for a way to obtain a digital diploma. She finds the respective page, which shows a link "request your digital diploma". She clicks on this link and is being sent to her digital wallet. The wallet notifies her that an issuer offered to issue a diploma credential. She confirms this inquiry and is being sent to the university's credential issuance service. She logs in with her university login and is being asked to consent to the creation of a digital diploma. She confirms and is sent back to her wallet. There, she is notified of the successful creation of the digital diploma.

## Issuer-Initiated Credential Issuance (Cross-Device / Credential Retrieval Only)

The user visits the administration office of her university to obtain a digital diploma. The university staff checks her student card and looks up her university diploma in the university's IT system. The office staff then starts the issuance process. The user is asked to scan a QR Code to retrieve the digital diploma. She scans the code with her smartphone, which automatically starts her wallet, where she is notified of the offer to create a digital diploma (the verifiable credential). She consents, which causes the wallet to obtain and store the verifiable credential.

## Issuer-Initiated Credential Issuance (with information pre-submitted by the End-User)

The user navigates to her university's webpage to obtain a digital diploma where she is asked to scan a QR Code to start the retrieval process. She scans the code with her smartphone, which automatically starts her wallet, where she is notified of the prerequisite to enter a PIN code for security reasons. This code was sent as a text message to her smartphone in the meantime. She enters the PIN and confirms the credential issuance, which causes the wallet to obtain and store the verifiable credential.

## Deferred Credential Issuance

The user wants to obtain a digital criminal record certificate. She starts the journey in her wallet and is sent to the issuer service of the responsible government authority. She logs in with her eID and requests the issuance of the certificate. She is notified that the issuance of the certificate will take a couple of days due to necessary background checks by the authority. She confirms and is sent back to the wallet. The wallet shows a hint in the credential list indicating that issuance of the digital criminal record certificate is under way. A few days later, she receives a notification from her wallet app telling her that the certificate was successfully issued. She opens her wallet, where she is asked after startup whether she wants to download the certificate. She confirms and the new credential is retrieved and stored in her wallet.

# Requirements

This section describes the requirements this specification aims to fulfill beyond the use cases described above. 

* Proof of possession of key material
  * Support all kinds of proofs (e.g., signatures, blinded proofs) but also issuance w/o proof
  * The proof mechanisms shall be complementary to OAuth/OpenID Connect mechanisms for request signatures, client authentication, and PoP to allow for its parallel usage.
* It shall be possible to request a single credential as well to request multiple credentials in the same request. Examples of the latter include: 
  * credentials containing different claims for the same user (micro/mono credentials) bound to the same key material
  * batch issuance of multiple credentials of the same type bound to different key material (see mDL)
* It shall be possible to issue multiple credentials based on same consent (e.g., different formats and/or keys - `did:key` followed by `did:ebsi`)
* Support for deferred issuance of credentials
* User authentication and identification
  * Issuer shall be able to dynamically obtain further data and be able to authenticate the user at their discretion
  * Application used by the End-User shall be able to pass existing credentials (as presentations) or identity assertions to the issuance flow
    * Assisted flow (utilizing credential manifest)
    * Presentations/assertions must be protected against replay
* It shall be possible to request standard OpenID Connect claims and credentials in the same flow (to implement wallet onboarding, see EBSI/ESSIF onboarding)
* Support for Credential metadata (Application used by the End-User shall be able to determine the types of credentials an issuer is able to issue)
* Ensure OAuth 2.0 Authorization Server is authoritative for respective credential issuer (OP (OpenID Connect issuer URL) <-> Issuer ID (DID))
* Incorporate/utilize existing specs
  * W3C VC HTTP API(?)
  * DIF Credential manifest(?)

# Overview 

This specification defines the following mechanisms to allow wallet applications (acting as OAuth and Credential Issuance API clients) used by the End-User to request credential issuers (acting as OAuth Authorization Servers and Credential Issuance API providers) to issue Verifiable Credentials via the Credential Issuance API:

* An optional mechanism to pre-obtain a Credential Manifest
* An extended authorization request syntax that allows to request credential types to be issued
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
        |                |          (1) initiate issuance                       |
        |                |<-----------------------------------------------------|        
        |    interacts   |                                                      |
        |--------------->|                                                      |
        |                |  (2) [opt] obtain credential manifest                |
        |                |----------------------------------------------------->|
        |                |              credential manifest                     |
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

This flow is based on OAuth and the code grant type. Use with other grant types, such as CIBA, will be possible as well. 

The starting point is an interaction of the user with her wallet. The user might, for example,

* want to present a credential and found out there is no suitable credential present in her wallet or
* have visited the web site of a Credential Issuer and wants to obtain a credential from that issuer. 

(1) (OPTIONAL) The issuer sends a request to the wallet to initiate the issuance flow. This request contains information about the 
credential(s) the End-User wants to obtain from that issuer, e.g., in the form of credential manifest IDs or credential types, and 
further data, e.g., hints about the user when the user is already logged in with the Issuer.

(2) (OPTIONAL) obtain credential manifest (as defined in [@DIF.CredentialManifest]) from the issuer with an information of which Verifiable Credentials the Issuer can issue, and optionally what kind of input from the user the Issuer requires to issue that credential.

Note: The wallet MAY also obtain the information about the credential issuer's capabilities using other means, which is out of scope of this specification. 

(4) In this step, the wallet sends an authorization request to the issuer. This request determines
the types of verifiable credentials the wallet (on behalf of the user) wants to obtain.

The wallet SHOULD use a pushed authorization request (see [@!RFC9126]) to first send the payload of 
the authorization request to the issuer and subsequently use the `request_uri` returned by the issuer in the authorization
request. This ensures integrity and confidentiality of the request data and prevents any issues raised by URL length restrictions 
regarding the authorization request URL.

Note: Signed and encrypted request objects would also ensure integrity and confidentiality. However, this approach would further
increase the URL size, which might decrease robustness of the process. 

The issuer takes over user interface control at this point and interacts with the user. The implementation of 
this step is at the discretion of the issuer.  

(4.1)  The issuer will typically authenticate the user in the first step of this process. For this purpose,
the issuer might use a local or federated login, potentially informed by an `id_token_hint` (see [@OpenID.Core]).

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

# Endpoints {#endpoints}

## Overview

This specification defines new endpoints as well as additional parameters to existing OAuth endpoints required to implement the protocol outlined in the previous section. It also introduces a new authorization details type according to [@!I-D.ietf-oauth-rar] to convey the details about the credentials the wallet wants to obtain. Aspects not defined in this specification are expected to follow [@!RFC6749]. it is RECOMMENDED to use PKCE as defined in [@!RFC7636] to prevent authorization code interception attacks.

There are the following new endpoints: 

* Issuance Initiation Endpoint: An endpoint exposed by the wallet that allows an issuer to initiate the issuance flow
* Credential Endpoint: this is the OAuth-protected API to issue verifiable credentials
* Deferred Credential Endpoint: this endpoint is used for deferred issuance of verifiable credentials 

The following endpoints are extended:

* Client Metadata: new metadata parameter is added to allow a wallet (acting as OAuth client) to publish its issuance initiation endpoint.
* Server Metadata: New metadata parameters are added to allow the client to determine what types of verifiable credentials a particular OAuth 2.0 Authorization Server is able to issue along with additional information about formats and prerequisites.
* Authorization Endpoint: The `authorization_details` parameter is extended to allow clients to specify types of the credentials when requesting authorization for issuance. These extension can also be used via the Pushed Authorization Endpoint, which is recommended by this specification. 
* Token Endpoint: optional parameters are added to the token endpoint to provide the client with a nonce to be used for proof of possession of key material in a subsequent request to the credential endpoint. 

## Client Metadata 

This specification defines the following new Client Metadata parameter in addition to [@!RFC7591] for wallets acting as OAuth client:

* `initiate_issuance_endpoint`: OPTIONAL. URL of the issuance initation endpoint of a wallet. 

If the issuer is unable to perform discovery of the Issuance Initiation Endpoint URL, the following static URL is used: `openid_initiate_issuance:`.

## Server Metadata

This section extends the server metadata [@!RFC8414] to allow the RP to obtain information about the credentials an OP is capable of issuing.

This specification defines the following new Server Metadata parameters for this purpose:

* `credential_endpoint`: REQUIRED. URL of the OP's Credential Endpoint. This URL MUST use the `https` scheme MAY contain port, path and query parameter components.

* `credentials_supported`: REQUIRED. A JSON object containing a list of key value pairs, where the key is a globally unique string serving as an abstract identifier of the credential. The value can be a JSON object or a URL of a page that contains a JSON object. The JSON object MUST conform to the structure of the (#credential-metadata-object). It communicates the specifics of the credential that the issuer support issuance of.

### Credential Metadata Object {#credential-metadata-object}

This section defines the structure of the object that appears as the value to the keys inside the object defined for the `credentials_supported` metadata element.

* `display`: OPTIONAL. An array of objects containing information how to display a certain credential in a wallet in each language. The following is a non-exhaustive list of parameters that MAY be included. Note that the display name of the credential is obtained from `display.name` and individual claim names from `claims.display` values.
  * `name`: REQUIRED. String value of a display name for the credential.
  * `language`: OPTIONAL. String value that identifies language of this diplay object. Multiple `diplay` objects may be included for separate languages. There MUST be only one object with the same language identifier
  * `credential_issuer`: OPTIONAL. String value of a display name for the credential issuer.
  * `background_color`: OPTIONAL. String value of a background color of the credential.
  * `text_color`: OPTIONAL. String value of a text color of the credential.
  * `logo`: OPTIONAL. A JSON object with information about the logo of the credential issuer with a following non-exhaustive list of parameters that MAY be included:
    * `url`: OPTIONAL. URL where the wallet can obtain a logo of the credential issuer.
    * `alternative_text`: OPTIONAL. String value of an alternative text of a logo image.
  * `description`: OPTIONAL. String value of a description of the credential.

* `formats`: REQUIRED. A JSON object containing a list of key value pairs, where the key is a string identifying the format of the credential. Below is a non-exhaustive list of valid key values defined by this specification
  * Claim Format Designations defined in [@!DIF.PresentationExchange], such as `jwt_vc` and `ldp_vc`
  * `mdl_iso`: defined in this specification to express a mobile driving licence (mDL) credential compliant to a data model and data sets defined in ISO/IEC 18013-5:2021 specification. 
  * `ac_vc`: defined in this specificaiton to express an AnonCreds credential format defined as part of the Hyperledger Indy project [Hyperledger.Indy].

The value in a key value pair is a JSON object detailing the specifics about the support for the credential format with a following non-exhaustive list of parameters that MAY be included:
  * `types`: REQUIRED. Array of strings representing a format specific type of a credential. This value corresponds to `type` in W3C [@!VC_DATA] and a `doctype` in ISO/IEC 18013-5 (mobile Driving License).
  * `cryptographic_binding_methods_supported`: OPTIONAL. Array of case sensitive strings that identify how the credential is bound to the identifier of the End-User who possesses the credential as defined in (#credential-binding). A non-exhaustive list of valid values defined by this specification are `did`, `mso`, and `none`.
  * `cryptographic_suites_supported`: OPTIONAL. Array of case sensitive strings that identify the cryptographic suites that are supported for the `cryptographic_binding_methods_supported`. Cryptosuites for credentials in `jwt_vc` format should use algorithm names defined in [IANA JOSE Algorithms Registry](https://www.iana.org/assignments/jose/jose.xhtml#web-signature-encryption-algorithms). Cryptosuites for credentials in `ldp_vc` format should use signature suites names defined in [Linked Data Cryptographic Suite Registry](https://w3c-ccg.github.io/ld-cryptosuite-registry/).

* `claims`: REQUIRED. A JSON object containing a list of key value pairs, where the key identifies the claim offered in the credential. The value is a JSON object detailing the specifics about the support for the claim with a following non-exhaustive list of parameters that MAY be included:
  * `namespace`: OPTIONAL. String value of a namespace that the claim belongs to. Relevant for ISO/IEC 18013-5 (mobile Driving License) specification.
  * `mandatory`: OPTIONAL. Boolean which when set to `true` indicates the claim MUST be present in the issued credential. If the `mandatory` property is omitted its default should be assumed to be `false`.
  * `value_type`: OPTIONAL. String value determining type of value of the claim. A non-exhaustive list of valid values defined by this specification are `string`, `number`, and image media types such as `image/jpeg` as defined in IANA media type registry for images (https://www.iana.org/assignments/media-types/media-types.xhtml#image).
  * `display`: OPTIONAL. String value of a display name for the claim.

It is dependent on the credential format, where the requested claims will appear.

The following example shows a non-normative example of the relevant entries in the OP metadata defined above

```
  HTTP/1.1 200 OK
  Content-Type: application/json

 {
  "credential_endpoint": "https://server.example.com/credential",
  "credentials_supported": {
    "546b92be-ac23-47cd-a97c-2d1a1736aedf" : {
      "display": [
        {
          "name": "University Credential",
          "credential_issuer": "Example University",
          "background_color": "#12107c",
          "text_color": "#FFFFFF",
          "logo": {
            "url": "https://exampleuniversity.com/public/logo.png",
            "alternative_text": "a square logo of a university"
          },
          "language": "en"
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
          "given_name": {},
          "last_name": {},
          "degree": {},
          "gpa": {
            "mandatory": false,
            "value_type": "number",
            "display": "GPA"
          }
      }
    }
  }
```

Note: The Client MAY use other mechanisms to obtain information about the verifiable credentials that an Issuer can issue.

## Issuance Initiation Endpoint

This endpoint is used by an issuer in case it is already in an interaction with a user that wishes to initate a credential issuance. It is used to pass available 
information relevant for the credential issuance to ensure a convenient and secure process. 

### Issuance Initiation Request {#issuance_initiation_request}

The issuer (or any other party wishing to kickstart an issuance into a certain wallet) sends the request as a HTTP GET request or a HTTP redirect to the Issuance Initiation Endpoint URL. 

The following request parameters are defined: 

* `issuer`: REQUIRED. The issuer URL of the credential issuer, the wallet is requested to obtain one or more credentials from. 
* `credential_type`: CONDITIONAL. A JSON string denoting the type of the credential the wallet shall request. MUST be present if `manifest_id` is not present.
* `manifest_id`: CONDITIONAL. A JSON String  refering to a credential manifests published by the credential issuer. MUST be present if `credential_type` is not present. 
* `op_state`: OPTIONAL. String value created by the Credential Issuer and opaque to the wallet that is used to bind the sub-sequent authentication request with the Credential Issuer to a context set up during previous steps. If the client receives a value for this parameter, it MUST include it in the subsequent Authentication Request to the Credential Issuer as the `op_state` parameter value.  

The following is a non-normative example:

```
  GET /initiate_issuance?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &op_state=eyJhbGciOiJSU0Et...FYUaBy
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

## Authorization Endpoint

The Authorization Endpoint is used in the same manner as defined in [@!RFC6749] taking into account the recommendations given in [@!I-D.ietf-oauth-security-topics] and utilizes [@!I-D.ietf-oauth-rar].

In addition to the required basic Authorization Request, this section also defines how pushed authorization requests can be used to protect the authorization request payload and when the requests become large.

### `authorization_details` Request Parameter {#authorization-details}

Request parameter `authorization_type` defined in Section 2 of [@!I-D.ietf-oauth-rar] MUST be used to convey the details about the credentials the wallet wants to obtain. This specification introduces a new authorization details type `openid_credential` and defines the following elements to be used with this authorization details type:

* `type` REQUIRED. JSON string that determines the authorization details type. MUST be set to `openid_credential` for the purpose of this specification.
* `credential_type`: CONDITIONAL. JSON string denoting the type of the requested credential. MUST be present if `manifest_id` is not present.
* `manifest_id`: CONDITIONAL. JSON String referring to a credential manifest published by the credential issuer. MUST be present if `type` is not present.
* `format`: OPTIONAL. JSON string representing a format in which the credential is requested to be issued. Valid values defined by this specification are `jwt_vc` and `ldp_vc`. Profiles of this specification MAY define additional format values.
* `locations`: OPTIONAL. An array of strings that allows a client to specify the location of the resource server(s) allowing the AS to mint audience restricted access tokens. This data field is predefined in Section 2.2 of ([@!I-D.ietf-oauth-rar]).

Note: `credential_type` and `format` are used when the Client has not pre-obtained a Credential Manifest. `manifest_id` is used when the Client has pre-obtained a Credential Manifest. These two approaches MAY be combined in one request in different authorization details objects.

Note: The `credential_application` element defined in [@DIF.CredentialManifest] is not required by this specification.

Note: Passing the `format` to the authorization request is informational and allows the credential issuer to refuse early in case it does not support the requested format/credential combination. The client MAY request issuance of credentials in other formats as well later in the process at the credential endpoint.

[TBD: `locations` could enable a single authorization server to authorize access to different credential endpoints. Might be an architectural option we want to pursue.]

A non-normative example of an `authorization_details` object. 

```json=
{
   "type":"openid_credential",
   "credential_type":"https://did.example.org/healthCard",
   "format":"ldp_vc"
}
```
Note: applications MAY combine `openid_credential` with any other authorization details type in an authorization request.
 
### Credential Authorization Request {#credential-request}

A credential authorization request is an OAuth Authorization request as defined in section 4.1.1 of [@!RFC6749], which requests to grant access to the credential endpoint as defined in (#credential-endpoint). It also follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

There are two possible ways to make a credential authorization request. One way is to use of the `authorization_details` request parameter as defined in (#authorization-details) with one or more authorization details objects of type `openid_credential`. The other is through the use of scopes as defined in (#credential-request-using-type-specific-scope).

A non-normative example of a credential authorization request using the `authorization_details` parameter  (uses PKCE as defined in [@!RFC7636]) (with line wraps within values for display purposes only).

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

This non-normative example shows a credential authorization request which is also a OpenID Connect authentication request  (uses PKCE as defined in [@!RFC7636]).

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

An alternative credential request syntax to that defined in (#credential-request) involves using an OAuth2 scope following the syntax defined below.

```
openid_credential:<credential-type>
```

The value of `<credential-type>` indicates the type of credential being requested, providers who do not understand the value of this scope in a request MUST ignore it entirely. The presence of a scope following this syntax in the request MUST be interpreted by the provider as a request for access to the credential endpoint as defined in (#credential-endpoint) for the specific credential type. Multiple occurrences of this scope MAY be present in a single request whereby each occurrence MUST be interpreted individually.

A non-normative example of a credential request scoped to a specific credential type (uses PKCE as defined in [@!RFC7636]).

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

This specification defines the following additional request parameters that can be supplied in any credential authorization request:

* `wallet_issuer`: OPTIONAL. JSON String containing the wallet's OpenID Connect Issuer URL. The Issuer will use the discovery process as defined in [@!SIOPv2] to determine the wallet's capabilities and endpoints. RECOMMENDED in Dynamic Credential Request.
* `user_hint`: OPTIONAL. JSON String containing an opaque user hint the wallet MAY use in sub-sequent callbacks to optimize the user's experience. RECOMMENDED in Dynamic Credential Request.
* `op_state`: OPTIONAL. String value identifying a certain processing context at the credential issuer. A value for this parameter is typically passed in an issuance initation request from the issuer to the wallet (see ((#issuance_initiation_request)). This request parameter is used to pass the `op_state` value back to the credential issuer. 

Note: When processing the authorization request, the issuer MUST take into account that the `op_state` is not guaranteed to originate from this issuer. It could have been injected by an attacker. 

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

The Issuer MUST utilize [@OIDC4VP] and [@!SIOPv2] to dynamically request additional credentials. From a protocol perspective, the Issuer acts now as a verifier and sends a presentation request to the wallet. The Client MUST have these credentials obtained prior to initiating a transaction with this Issuer. 

This provides the benefit of the Issuer being able to adhere to the principle of data minimization, for example by including only minimum requirements in the Credential Manifest knowing that it can supplement additional information if needed.

To enable dynamic callbacks of the issuer to the end-user's wallet, the wallet MAY provide additional parameters `wallet_issuer` and `user_hint` defined in the Authorization Request section of this specification.

For non-normative examples of request and response, see section 11.6 in [@OIDC4VP].

Note to the editors: need to sort out credential issuer's client_id with wallet and potentially add example with `wallet_issuer` and `user_hint` 

### Successful Authorization Response

Authentication Responses MUST be made as defined in [@!RFC6749].

Below is a non-normative example of a successful Authentication Response:

```
HTTP/1.1 302 Found
  Location: https://wallet.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
```

### Authentication Error Response

Authentication Error Response MUST be made as defined in [@!RFC6749].

The following is a non-normative example of an unsuccessful token response.

```json=
HTTP/1.1 302 Found
Location: https://client.example.net/cb?
    error=invalid_request
    &error_description=Unsupported%20response_type%20value
```

## Token Endpoint

The Token Endpoint issues an Access Token and, optionally, a Refresh Token in exchange for the authorization code that client obtained in a successful Authorization Response. It is used in the same manner as defined in [@!RFC6749] and follows the recommendations given in [@!I-D.ietf-oauth-security-topics].

### Token Request

Upon receiving a successful Authentication Response, a Token Request is made as defined in Section 4.1.3 of [@!RFC6749].

Below is a non-normative example of a token request:
```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
  grant_type=authorization_code
  &code=SplxlOBeZQQYbYS6WxSbIA
  &code_verifier=dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk
  &redirect_uri=https%3A%2F%2Fwallet.example.org%2Fcb
  
```

### Successful Token Response

Token Requests are made as defined in [@!RFC6749].

In addition to the response parameters defined in [@!RFC6749], the AS MAY return the following parameters:

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
    "c_nonce": "tZignsnFbp",
    "c_nonce_expires_in": 86400
  }
```

### Token Error Response

If the Token Request is invalid or unauthorized, the Authorization Server constructs the error response as defined as in Section 5.2 of OAuth 2.0 [@!RFC6749].

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

### Binding the Issued Credential to the identifier of the End-User possessing that Credential {#credential-binding}

Issued credential SHOULD be cryptographically bound to the identifier of the End-User who possesses the credential. Cryptographic binding allows the Verifier to verify during presentation that the End-User presenting a credential is the same End-User to whom it was issued. For non-cryptographic type of binding and credentials issued without any binding, see Implementations Considerations sections {#claim-based-binding} and {#no-binding}. 

Note that claims in the credential are usually about the End-User who possesses it, but can be about the other entity.

For cryptographic binding, the Client has the following options to provide cryptographic binding material for a requested credential as defined in {#credential_request}:

1. Provide proof of control alongside key material (`proof` that includes `sub_jwk` or `did`)
1. Provide only proof of control without the key material (`proof` that does not include `sub_jwk` or `did`)

For more details, see {#did-binding} in the Security Considerations Section.

### Credential Request {#credential_request}

A Client makes a Credential Request by sending a HTTP POST request to the Credential Endpoint with the following parameters:

* `type`: REQUIRED. Type of credential being requested. It corresponds to a `schema` property in a Credential Manifest obtained in a setup phase.
* `format`: OPTIONAL. Format of the credential to be issued. If not present, the issuer will determine the credential 
format based on the client's format default.
* `proof` OPTIONAL. JSON Object containing proof of possession of the key material the issued credential shall be 
bound to. The `proof` object MUST contain the following `proof_type` element which determines its structure:

  * `proof_type`: REQUIRED. JSON String denoting the proof type. 

This specification defines the following values for `proof_type`:

* `jwt`: objects of this type contain a single `jwt` element with a signed JWT as proof of possession. The JWT MUST contain the following elements:
    * `kid`: CONDITIONAL. JWT header containing the key ID. If the credential shall be bound to a DID, the `kid` refers to a DID URL which identifies a particular key in the DID Document that the credential shall be bound to.
    * `jwk`: CONDITIONAL. JWT header containing the key material the new credential shall be bound to. MUST NOT be present if `kid` is present.
    * `iss`: REQUIRED. MUST contain the client_id of the sender
    * `aud`: REQUIRED. MUST contain the issuer URL of credential issuer
    * `iat`: REQUIRED. MUST contain the instant when the proof was created
    * `nonce`: REQUIRED. MUST contain a fresh nonce as provided by the issuer

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

# Pre-Authorized Code Flow

This section specifies an additional flow to obtain an access token for credential issuance. It is intended to support scenarios where the user starts a process on an issuer's website that ultimately results in one or more credentials being issued to the user's wallet. The process on the issuer's website may include uploading documents and presenting verifiable credentials to the issuer. Moreover, the End-user may be accessing the issuer's website on a device different from the one with the wallet application so the credential issuance process needs to be transfered to another device where the user's wallet resides. 

In contrast to the flow specified in (#endpoints), this flows is initiated by the Issuer when the credentials are "ready" and need to be "picked up" by the wallet application. How the user provides information required for the issuance of a requested credential to the Issuer and the business processes conducted by the Issuer to issue a credential are out of scope of this specification.

1. To initiate the flow, the Issuer needs to communicates to the wallet two parameters: the pre-authorized code and the credential type the code is valid for. The Issuer can send them in two different ways. Either the issuer sends them in an issuance initiation request to the Issuance Initiation Endpoint of the wallet as described in (#issuance_initiation_request). Or the issuer renders a QR code containing the issuance initiation request with these parameters. In this case, the user scans the QR code with her wallet on a different device in order to proceed. 
1. When initiating the flow, the Issuer also indicates if the PIN is required. If it is, sends PIN to the wallet using a channel different from an issuance initiation request.
1. The wallet sends the pre-authorized code to the issuer's Token Endpoint. This request MUST contain a user PIN if requested by the issuer. 
1. The issuer responds with an access token valid for credential issuance. 
1. The wallet sends a credential issuance request to the credential endpoint as defined in (#credential_request) using the credential type as indicated in the first step from the issuer to the wallet.
1. The issuer returns the requested credential as defined in (#credential_response). 

Steps 1 through 3 constitute a new kind of pre-authorized code flow that is implemented using an additional initiate issuance endpoint and a new OAuth grant type "urn:ietf:params:oauth:grant-type:pre-authorized_code". Steps 4 through 6 conform to the process specified in (#endpoints).

Note that the pre-authorized code is sent to the Token Endpoint and not to the Authorization Endpoint of the Issuer.

The following sections specify the extensions of a pre-authorized code flow.

## Extension Parameters for Initiate Issuance Request

These are the extension parameters defined for the pre-authorized code flow:

* `pre-authorized_code`: REQUIRED. The code representing the issuer's authorization for the wallet to obtain credentials of a certain type. This code MUST be short lived and single-use.
* `user_pin_required`: OPTIONAL. Boolean value specifying whether the issuer expects presentation of a user PIN along with the token request. Default is `false`. This PIN is intended to bind the pre-authorized code to a certain transaction in order to prevent replay of this code by an attacker that, for example, scanned the QR code while standing behind the legit user. 

Below is a non-normative example of an initiate issuance request:
```
  GET /initiate_issuance?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
```

The contents of a QR code could look like this:
```
openid_initiate_issuance://?
    issuer=https%3A%2F%2Fserver%2Eexample%2Ecom
    &credential_type=https%3A%2F%2Fdid%2Eexample%2Eorg%2FhealthCard 
    &pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
    &user_pin_required=true
```

## Extension Parameters for Token Request

These are the extension parameters to the token request for the grant type `pre-authorized_code`:

* `pre-authorized_code`: REQUIRED. The code representing the authorization to obtain credentials of a certain type. 
* `user_pin`: OPTIONAL. String value containing a user PIN. This value MUST be present if `user_pin_required` was set to `true` in the Initiate Issuance Request. The string value MUST consist of maximum 8 numeric characters (the numbers 0 - 9).

Below is a non-normative example of a token request:
```
POST /token HTTP/1.1
  Host: server.example.com
  Content-Type: application/x-www-form-urlencoded
  Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW

  grant_type=urn:ietf:params:oauth:grant-type:pre-authorized_code
  &pre-authorized_code=SplxlOBeZQQYbYS6WxSbIA
  &user_pin=493536
```

## Extension to the token response {#pre-authz-token-response}

Upon receiving `pre-authorized_code`, the issuer MAY decide to interact with the end-user in the course of the token request processing, which might take some time. In such a case, the issuer SHOULD respond with the error `authorization_pending` and the new return parameter `interval`: 

* `authorization_pending`: OPTIONAL. The token request is still pending as the issuer is waiting for end user end user interaction to complete.  The client SHOULD repeat the token request. Before each new request, the client MUST wait at least the number of seconds specified by the "interval" response parameter.
* `interval`: OPTIONAL. The minimum amount of time in seconds that the client SHOULD wait between polling requests to the token endpoint.  If no value is provided, clients MUST use 5 as the default.

## Replay Prevention

The pre-authorized code flow is vulnerable to the replay of the pre-authorized code, because by design it is not bound to a certain device (as the authorization code flow does with PKCE). This means an attacker can replay at another device the pre-authorized code meant for a victime, e.g., the attacker can scan the QR code while it is displayed on the victim’s screen, and thereby getg access to the credential. Such replay attacks must be prevented using other means. The design facilitates the following options: 

* User PIN: the issuer might set up a PIN with the user (e.g. via text message or email), which needs to be presented in the token request.
* Callback to device where the transaction originated: upon receiving the token request, the issuer asks the user to confirm the originating device (device that displayed the QR code) that the issuer may proceed with the credential issuance process. While the issuer reaches out to the user on the other device to get confirmation, the issuer returns an `authorization_pending` error code to the wallet (as described in (#pre-authz-token-response)). The wallet is required to call the token endpoint again to obtain the access token. If the user does not confirm, the token request is returned with the `access_denied` error code. This flow gives the user on the originating device more control over the issuance process.

## PIN Code Phishing

An attacker might leverage the credential issuance process and the user's trust into the wallet to phish PIN codes sent out by a different service that grant attacker access to services other than credential issuance. The attacker could setup a credential issuer site and in parallel to the issuance request trigger transmission of a PIN code to the user's phone from a service other than credential issuance, e.g. from a payment service. The user would then be asked to enter this PIN into the wallet and since the wallet sends this PIN to the token endpoint of the issuer (the attacker), the attacker would get access to the PIN code, and access to that other service. 

In order to cope with that issue, the wallet is RECOMMENDED to interact with trusted issuers only. In that case, the wallet would not process an initiate issuance request with an untrusted issuer URL. The wallet MAY also show to the user the endpoint or issuer it will be sending the PIN code and ask the user for confirmation.

# Security Considerations

# Implementation Considerations

## Claim-based Binding of the Credential to the End-User possessing the credential {#claim-based-binding}

Credential not cryptographically bound to the identifier of the End-User possessing it (see (#credential-binding)), should be bound to the End-User possessing the credential based on the claims included in that credential. 

In claim-based binding, no cryptographic binding material is provided. Instead, the issued credential includes user claims that can be used by the Verifier to verify possession of the credential by requesting presentation of existing forms of physical or digial identification that includes the same claims (e.g., a driver's license or other ID cards in person, or an online ID verification service).

## Binding of the Credential without Cryptographic Binding nor Claim-based Binding {#no-binding}

Some Issuers might choose issuing bearer credentials without either cryptographic binding nor claim-based binding, because they are meant to be presented without proof of possession.

One such use-case is low assurance credentials such as coupons or tickets. 

Another use-case is when the Issuer uses cryptographic schemes that can provide binding to the End-User possessing that credential without explicit cryptographic material being supplied by the application used by that End-User. For example, in the case of the BBS Signature Scheme, the issued credential itself is a secret and only derivation of a credential is presented to the Verifier. Effectively, credential is bound to the Issuer's signature on the credential, which becomes a shared secret transferred from the Issuer to the End-User.

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

# IANA Considerations

register "urn:ietf:params:oauth:grant-type:pre-authorized_code"

# Acknowledgements {#Acknowledgements}

We would like to thank David Chadwick, John Bradley, Alen Horvat, Michael B. Jones, and David Waite for their valuable contributions to this specification.

# Notices

Copyright (c) 2022 The OpenID Foundation.

The OpenID Foundation (OIDF) grants to any Contributor, developer, implementer, or other interested party a non-exclusive, royalty free, worldwide copyright license to reproduce, prepare derivative works from, distribute, perform and display, this Implementers Draft or Final Specification solely for the purposes of (i) developing specifications, and (ii) implementing Implementers Drafts and Final Specifications based on such documents, provided that attribution be made to the OIDF as the source of the material, but that such attribution does not indicate an endorsement by the OIDF.

The technology described in this specification was made available from contributions from various sources, including members of the OpenID Foundation and others. Although the OpenID Foundation has taken steps to help ensure that the technology is available for distribution, it takes no position regarding the validity or scope of any intellectual property or other rights that might be claimed to pertain to the implementation or use of the technology described in this specification or the extent to which any license under such rights might or might not be available; neither does it represent that it has made any independent effort to identify any such rights. The OpenID Foundation and the contributors to this specification make no (and hereby expressly disclaim any) warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to this specification, and the entire risk as to implementing this specification is assumed by the implementer. The OpenID Intellectual Property Rights policy requires contributors to offer a patent promise not to assert certain patent claims against other contributors and against implementers. The OpenID Foundation invites any interested party to bring to its attention any copyrights, patents, patent applications, or other proprietary rights that may cover technology that may be required to practice this specification.

# Document History

   [[ To be removed from the final specification ]]

   -05

   * added support for pre-authorized code flow
   * changed base protocol to OAuth

   -04

   * added support for requesting credential authorization with scopes 
   * removed support to pass VPs in the authorization request
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
