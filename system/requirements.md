# Requirements

See our Rebooting Web of Trust VIII Paper [Designing a Recipient-Centric Educational Digital Credential Ecosystem](https://github.com/WebOfTrustInfo/rwot8-barcelona/blob/master/topics-and-advance-readings/educational-credentialing-ecosystem.md) for additional context on the origin of DCC's approach. That paper details the requirements behind DCC's decision to use Verifiable Credentials and Decentralized Identifiers. This borrows heavily from that paper, but is a living document describing DCC's requirements.


## Basic Requirements

The credentialing system has the following minimal set of requirements:

*   An issuance should require the consent of both the issuer and recipient
*   A recipient should be able to prove the credential was issued by the issuer without requiring interaction with the issuer
*   The system should minimize the risk of recipient correlation, for example:
    *   Avoid subject (recipient) identifier (e.g. DID) reuse
    *   Incorporate nonces/recipient signatures (to avoid matching on hashed or signed values)
*   The recipient should consent to use of their credential
    *   From Verifiable Credential Data Model: "[Holders](https://w3c.github.io/vc-data-model/#dfn-holders) are positioned between [issuers](https://w3c.github.io/vc-data-model/#dfn-issuers) and [verifiers](https://w3c.github.io/vc-data-model/#dfn-verifier) and use [verifiable credentials](https://w3c.github.io/vc-data-model/#dfn-verifiable-credentials) to make [verifiable presentations](https://w3c.github.io/vc-data-model/#dfn-verifiable-presentations) to [verifiers](https://w3c.github.io/vc-data-model/#dfn-verifier)."
    *   A side effect is that the recipient can not be forced to present a revoked credential
*   The issuer should be able to revoke the credential
    *   Note that in the VC ecosystem, recipient revocation is not necessarily a requirement, because a recipient won't be forced to present a credential they disavow
        *   Anti-correlation and other privacy measures help ensure the recipient can't be linked to the credential
*   Once revoked, a credential must not return as "verified" by a confirming verification process. However, the system must enable proof that the credential was once valid one for some time period
*   Revoking and changing signing keys (issuer or recipient) should not invalidate previously-issued valid credentials
*   Revocation check should not reveal any personally identifiable information (or any additional information about that credential or batch)


## New requirements (TODO: review and refine)

*   Credential encryption (at rest and in transit) must be enabled by default
*   Minimize recipient agent/infrastructure requirements. Any required tools must be free and open source
*   Flexible credential schema supporting degrees, microcredentials, transcripts, skills
*   Credential format suitable for credentials containing personal data
*   In addition to credential verification, enable means of recipient authentication; i.e. relying parties can validate that the recipient corresponds to the subject of the credential. 
*   Maintain flexibility for a range of identity providers and auth mechanisms, including self-sovereign approaches, on both issuer and recipient side. 
    *   Enable utility of credentials after graduation, e.g.  EDUROAM or Touchpoint, are not guaranteed to be usable by the student after graduation


Beyond this minimal list of functional requirements, we plan to update this with policy-level considerations -- such as environmental sustainability.

We expect to extend/refine this list of requirements, but our guiding heuristic is the solution must be "better than paper". Specifically, in the process of improving efficiency, portability, and interoperability, we must not threaten privacy and control of the individual in the process. In fact; ideally privacy measures and control of the individual will strictly improve. This goal requires more elaboration in the list of requirements.


## Simplifying Assumptions

*   Limited set of institutions initially
*   Those institutions are trusted to maintain a registry that contains a mapping of institution names to key material
*   Verifiers/Consumers/Relying parties trust the registry and know to consult it when presented with the credential

To enable verification that does not involve the issuer, the registry must be highly available and allow lookup of valid issuer keys for a given time range.

![Credential Ecosystem Design](credential_system.png)