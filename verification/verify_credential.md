# Verify Credential / Presentation

## Verification of Verifiable Presentations vs Verifiable Credentials

It's important to highlight that in general, verifiers won't encounter standalone VCs. They're going to be wrapped in VPs. Wrapping a verifiable credential in a presentation serves several important goals. One, it prevents a class of common replay attacks (by requiring the inclusion of `domain` and `challenge` fields) -- the verifier knows that this VP was created _in response to a particular challenge_, and created just for the use of this verifier. And two, depending on the use case, if a VP is signed, it can serve to help identify the _holder_, the entity that's handing over this VC. Which can be important in many use cases where the person handing over a credential has to be the person to whom it was issued in the first place.

## Verification Overview

Layered process

1. Verify the Verifiable Presentation
2. Verify the Verifiable Credential that's contained in the presentation.

### Verify Presentation

1. make sure it's well-formed, according to the VC Data Model spec
2. make sure the `domain` and `challenge` values are ones the verifier was expecting
3. if the VP is signed, verify the signature

### Verify Credential(s)

Another helpful way of thinking about the verification process is to divide it into generic VC validation (which will be handled by a VC library), and use-case-specific verification (where you apply the business logic of your particular domain) that needs to be done by your application.

#### Generic VC verification/validation

*Handled by any VC library*

1. Is it a well-formed VC (according to the VC Data Model spec), are all the required fields present, etc.
2. Check the 'expiration' and 'not before this date' fields.
3. Verify signature. This involves:
    - a. Cryptographic signature verification (indicates the credential has not been tampered with)
    - b. Resolving the issuer DID (it needs to be resolvable)
    - c. Check to make sure that the keys the VC was signed by are actually authorized for the purposes of signing. 

#### App-specific / Business Logic Validation

These steps are not  standardized at this point, but it's assumed that a verifier app will do these as well.

1. Check the issuer DID against an allow-list (that's our [Issuer registry](https://github.com/digitalcredentials/docs/blob/main/identity/issuer_registry.md) that is trusted by this particular verifier.
2. Perform any other app-specific or domain-specific validation.

#### Grey area (DID and key revocation)
This is logic that should theoretically be standardized / be common across various DIDs and VCs, but currently.. is not. Assume that we'll be writing this logic for our DCC MVP, in sign-and-verify-core.

1. Check to make sure the signing keys are not marked revoked in the DID document (or have not been removed).
     * If the keys _are_ currently revoked, try and determine whether they were valid at the time the VC was issued, and whether to accept it. (This is very much usecase/threat-model specific).
     * Different DID methods and Key types have different ways of marking key expiration/revocation.
2. Check to make sure the issuer DID has not been revoked.
     * Although this sounds standard, it'll very much depend on the DID method. Some DID Resolver libraries will automatically check DID revocation during resolution. Otherwise, this will need to be done in the verification app, since the VC library is unlikely to perform this step automatically.
3. Check the VC identifier against the appropriate revocation method / revocation registry. Again, this should theoretically be done by the VC library, but since this is a very early stage / developing area, but currently VC libraries may not actually do this.

### Additional Notes

- Expired/revoked keys are out of scope for DID spec v1, so it's up to the did method to implement it
- Alternatively, we could implement OB issuer profile sort of key lists, in the category of "App-specific / Business Logic Validation"
