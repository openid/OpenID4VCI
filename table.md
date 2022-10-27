
<! -- 

The following table defines how issued Credential MUST be returned in the `credential` claim in the Credential Response based on the Credential format and the signature scheme. This specification does not require any additional encoding when Credential format is already represented as a JSON object or a JSON string.

| Credential Signature Format | Credential Format Identifier | Signature Scheme | Need for encoding when returning in the Credential Response  |
|:------|:-----|:-----|:------------|
|JWS Compact Serialization | `jwt_vc`, `mdl_iso_json`, `mid_iso_json` | Credential conformant to the W3C Verifiable Credentials Data Model, ISO/IEC 18013-5:2021 mobile driving licence (mDL) data model, or ISO/IEC 23220-4 mobile eID document data model (not yet published), and signed as a JWS Compact Serialization. | MUST be a JSON string. Credential is already a sequence of base64url-encoded values separated by period characters and MUST NOT be re-encoded. |
|JWS JSON Serialization | `jwt_vc`, `mdl_iso_json`, `mid_iso_cbor`| Credential conformant to the W3C Verifiable Credentials Data Model, ISO/IEC 18013-5:2021 mobile driving licence (mDL) data model, or ISO/IEC 23220-4 mobile eID document data model (not yet published), and signed as a JWS JSON Serialization. | MUST be a JSON object. MUST NOT be re-encoded. |
|Data Integrity | `ldp_vc` | Credential conformant to the W3C Verifiable Credentials Data Model and signed with Data Integrity Proofs. | MUST be a JSON object. MUST NOT be re-encoded. |
| CL-Signatures |`ac_vc` | Credential conformant to the AnonCreds format as defined in the Hyperledger Indy project and signed using CL-signature scheme. | MUST be a JSON object. MUST NOT be re-encoded. |
| COSE |`mdl_iso_cbor`| Credential conformant to the ISO/IEC 18013-5:2021 mobile driving licence (mDL) data model, encoded as CBOR and signed as a COSE message. | MUST be a JSON string that is the base64url-encoded representation of the issued Credential |

Note that this table might be superceded by a registry in the future. Meanwhile, for interoperability, implementers MUST follow the requirements defined in the table above.

-->