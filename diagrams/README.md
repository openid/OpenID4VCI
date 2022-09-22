These sequence diagrams shall illustrate different design options to implement credential issuance in scenarios with substantial or high security security requirements. 

The focus is on the wallet design, the issuer design is out of scope. 

The design options described here are based on the security considerations in https://openid.bitbucket.io/connect/openid-4-verifiable-credential-issuance-1_0.html#name-trust-between-wallet-and-is. 

Most of the design options assume the wallet is an app running on a smartphone and utilizing the device's capabilities to secutely store keys and credentials. This app then uses a cloud service for some operations relevant for achieving the desired security level, namely app attestation and client authentication towards the issuer. The interface towards the issuer is always the client authentication based on trust established between the wallet provider and the issuer (directly or via a registry). 

One flow show how a web wallet could be implemented. 

All flows assume the different components are operated by the Wallet Provider.

Terms:

* (Wallet) Frontend: Wallet mobile application that the End-User interacts with.
* (Wallet) Backend: Component of the Wallet in the form of a Cloud service. 

Options:

* __attestation_native_app_bff.plantuml__ - The native app utilizes the cloud service according to the "Backend for Frontend" pattern that is common practice in the OAuth world. It means, in particular, that the app relies on its cloud service to authenticate towards the issuer (PAR & token endpoint) to set up a pushed authorization request and to get the access token to obtain one or more credentials. To maximize privacy, the actual credential issuance is NOT sent through the backend. Instead the wallet directly calls the issuer's credential endpoint. A variant of this option for issuer initiated issuance is given in attestation_native_app_issuer_initiated_bff.plantuml
* __attestation_native_app_client_assertion.plantuml__ - The native app utilizes it's cloud service to get a short living assertion that it then uses to directly authenticate with the credental issuer (PAR & token endpoints) to set up a pushed authorization request and obtain the access token from the issuer.  A variant of this option for issuer initiated issuance is given in attestation_native_app_issuer_initiated_client_assertion.plantuml
* __attestation_native_app_cert.plantuml__/__attestation_native_app_cert_provisioning.plantuml__ - every native app instance is provisioned with a X.509 certificate by its cloud service, i.e. the wallet provider is the CA for those certificates. Such a certificate is used to authenticate the native app towards the credential issuer (PAR & token endpoint). The issuer must verify that the certificate was issued by a trusted party (the wallet provider). 