# Threat model

See our Rebooting Web of Trust VIII Paper [Designing a Recipient-Centric Educational Digital Credential Ecosystem](https://github.com/WebOfTrustInfo/rwot8-barcelona/blob/master/topics-and-advance-readings/educational-credentialing-ecosystem.md) for additional context on the origin of DCC's approach. This borrows heavily from that paper, but is a living document describing DCC's threat model.

## Must handle

The system should handle the following threats:

*   A recipient lies about the content of a credential they received 
*   A recipient lies about the existence of a (not existing) credential
*   A recipient tampers with the display of the credential they received


## Unknown

Ideally the system should handle these threats, but this forces certain implementation choices. These will be elaborated on in the "Analysis" section:

*   Issuer cannot lie about date of issue
*   Malicious actors inside valid issuer institution issues credentials that should not exist

### Issuance Date

To clarify, the date of issue this may differ from the date of award. The issuing institution must be able to backdate awards recieved in the past. The problem here is whether the issuance date timestamp stated by the issuer in the credential can be trusted. 

Trusted timestamping can be used to support issuer claims of issuance date (or replace). This can be accomplished by a range of methods including TSAs, blockchain anchoring, and Open Timestamps. 

At the same time, the current VC threat model does not call out this threat, which is why we raise this here.

### Fraudulent issuance detection

This threat involves a bad actor in the issuing institution, who is able to sign with the issuing address, that is issuing fraudulent credentials to friends/collaborators. Does a verification process that cuts out the issuer increase this risk?

One benefit of blockchain-anchoring is that one can alert on issuer addresses to detect suspicious issuances. However, if batch issuance is used, (and the issuer is not involved in verification), it may not be possible to detect fraudulent entries without additional metadata or an audit trail of the batch recipients (e.g. if the issuer records who is expected in the batch, and can deterministically reproduce the batch, they can verify that the merkle root matches what's recorded on chain).

Which level of the credential issuance solution should handle this threat? And do VC threat models call proper attention to this risk?

While this can be solved on a case-by-case basis, we think this problem needs broader awareness. 

## Out of scope [TODO]

*   The system does not prevent illegitimate issuing organizations, e.g. "Fake University of North South Idaho". Other existing mechanisms exist to police them, e.g. accreditation bodies, federal funding guidelines, etc...
*   The system cannot determine whether a backdated credential is valid (e.g. it must allow generation of proof for an individual that graduated 5 years ago)
*   The system cannot determine exact time of issuance; it can only determine the (approx) time of issuance via "trusted" timestamps (blockchain anchoring and open timestamps)


## In-scope threats, by actor

*   Issuer
    *   Malicious issuer may revoke a credential, but observers can see it was once valid
    *   Malicious issuer may fraudulently issue a credential, but the issuance should be detectable (auditable)
*   Recipient
    *   Malicious recipient may obtain a fake credential, but the issuance should be detectable (auditable)


## Detecting a malicious party in the issuing organization

It's straightforward to mitigate/detect this problem as part of a broader system deployment, e.g. audit logs in a SaaS product. But that system must similarly take measures to prevent or detect malicious actors/agents. So this is just a special case of a broader set of fraudulent issuance concerns. 

Solutions may involve a range of tactics in a deployed system, including:
- Address monitoring (if blockchain-anchored)
- Audit logs
- Improved credential provenance measures 

In earlier thinking, we latched onto the ability to perform address monitoring on blockchain-anchored credentials as necessary 

## Improved Credential Provenance
In this model, the credential would not just say "created and signed by MIT", but also:
- the human actor that triggered the credential creation and signature
- their authority to do so on MIT's behalf
- the system on which it occurred and the version of software used, etc. 
- timestamps associated with actions

This delegation tracking is similar to why bug reports need to know the user's OS, browser, and versions of both. Unless you characterize the system used to generate the credential, you only have the trust in key management of the issuing entity for evaluating the quality of the credential. 

For example: it can be a fairly simple trojan horse that wraps itself around the signing app to get inside the signing machine and generate false signatures. The user unwittingly uses the trojan horse instead of the real application, including the authentication into the private keys. This is why some of the IoT credential assurance procedures require using a machine that has never been and never will be on the network--only an isolated device meets the standard. 

Further, the last bullet allows an institution to revoke a person's ability to generate valid credentials without revoking credentials properly issued by that same person. 

The set of requirements outlined here do not address this kind of provenance. 

The added provenance also allows verifiers to spot known compromised software packages and request the holder get a fresh credential. 


## References

*   Verifiable Credentials Use Case and Requirements: https://w3c.github.io/vc-data-model/#use-cases-and-requirements
*   VC Data Model: [https://w3c.github.io/vc-data-model/](https://w3c.github.io/vc-data-model/)
*   VC Data Model Explainer: [https://github.com/w3c/vc-data-model/blob/gh-pages/VCDMExplainer.md](https://github.com/w3c/vc-data-model/blob/gh-pages/VCDMExplainer.md)
*   DID Primer: [https://github.com/WebOfTrustInfo/rwot5-boston/blob/master/topics-and-advance-readings/did-primer.md](https://github.com/WebOfTrustInfo/rwot5-boston/blob/master/topics-and-advance-readings/did-primer.md)

