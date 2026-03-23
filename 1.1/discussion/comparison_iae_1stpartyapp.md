|       | [Interactive Authorization Endpoint (VCI 1.1 current draft)](https://openid.github.io/OpenID4VCI/openid-4-verifiable-credential-issuance-1_1-wg-draft.html#name-interactive-authorization-e) | [Oauth 2.0 for 1st Party Application (Draft-3)](https://www.ietf.org/archive/id/draft-ietf-oauth-first-party-apps-03.html)  | Open Questions? |
|-------|------------------------------------|--------------------------------------|---------|
| **security model**      |  Third-party application. Wallet is not controlled by same entity as authorization server that protects the credential issuer                                 |   First-party applications are applications that are controlled by the same entity as the authorization server. (Profiles can use is for third-party applications with risk assessment)  |  What risks are associated with third-party apps directly interacting with the user?         |
| **endpoint**      |  `interactive_authorization_endpoint` parameter in AS Metadata                            |    `authorization_challenge_endpoint` parameter in AS Metadata                                    |         |
| **requests**      |  (Initial and Follow-Up) Interactive Authorization Request                                    |      Authorization Challenge Request, Intermediate Requests are out of scope and must be defined by profile, `authorization_challenge_endpoint` can be reused)                               |         |
| **responses**      |   Interaction Required Response, Authorization Code Response, Interactive Authorization Error Response                                  | Authorization Code Response, Error Response (includes requests for more information)                                    |         |
| **scope**      | implementation ready specification that defines two interaction types (`urn:openid:dcp:iae:openid4vp_presentation`, `urn:openid:dcp:iae:redirect_to_web`)                                   | defines basic mechanics + `redirect_to_web` option. In order to successfully implement this specification, the Authorization Server will need to define its own specific profile
| **redirect_to_web**      | `redirect_to_web` can be used multiple times in any place in Authorization process. As returns code (finished) or access code (follow up)                                 | `redirect_to_web` as fallback (initiates a new authorization code flow). Not described how to proceed interaction after redirect to client                                       |  is this a limitation, that should be solved? |                                     |         |
| **extensibility**      |  can be extended by interaction_types (collision-resistant URNs)                                 |  can be extended by profiles (no explicit profile name needed in request) which defines additional parameters and error codes                                    |  can we build interaction_types in profile or should this part of the spec?        |
| **negotiation**      |  by using interaction_types_supported in initial request                                 |  not defined ([#173](https://github.com/oauth-wg/oauth-first-party-apps/issues/173))                           |  see above        |
| **auth_session**      | MUST be distinct for each interactive authorization response. DPOP binding under discussion ([#718](https://github.com/openid/OpenID4VCI/issues/718)). Not (re)used by client for new initial request.                                   |  SHOULD be associated with DPOP public, MUST be stored by client beyond the issuance of the authorization code ([#174](https://github.com/oauth-wg/oauth-first-party-apps/issues/174))                                    |         |
|       |                                    |                                      |         |
|       |                                    |                                      |         |
|       |                                    |                                      |         |
|       |                                    |                                      |         |


## OpenID4VCI Profile Draft

- [ ] describe how the application of this specification avoids the risks associated with third-party apps directly interacting with the user
- [ ] custom Authorization Challenge Request Parameters
- [ ] custom Authorization Challenge Response Parameters
- [ ] additional requests needed?

### Authorization Challenge Request Parameters
In addition to the request parameters defined in Section 5.1, the authorization server defines the additional parameters below.

"_":
??

### Authorization Challenge Response Parameters
In addition to the response parameters defined in Section 5.2, the authorization server defines the additional parameter below.

"_":
??