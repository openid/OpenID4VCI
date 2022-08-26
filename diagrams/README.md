These sequence diagrams shall illustrate different design options to implement credential issuance in scenarios with substantial or high security security requirements. 

The focus is on the wallet design, the issuer design is out of scope. 

The design options described here are based on the security considerations in https://openid.bitbucket.io/connect/openid-4-verifiable-credential-issuance-1_0.html#name-trust-between-wallet-and-is. 

Most of the design options assume the wallet is an app running on a smartphone and utilizing the device's capabilities to secutely store keys and credentials. This app then uses a cloud service for some operations relevant for achieving the desired security level, namely app attestation and client authentication towards the issuer. The interface towards the issuer is always the client authentication based on trust established between the wallet provider and the issuer (directly or via a registry). 

One flow show how a web wallet could be implemented. 

All flows assume the different components are operated by the Wallet Provider.

Terms:

* (Wallet) Frontend: Wallet mobile application that the End-User interacts with.
* (Wallet) Backend: Component of the Wallet in the form of a Cloud service. 