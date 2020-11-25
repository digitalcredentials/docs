# Identity Design Decisions

DCC uses [Verifiable Credential Data Model](https://w3c.github.io/vc-data-model/), which provides a flexible framework for issuer and learner identification via URIs, enabling the use of traditional identity schemes as well as new emerging decentralized or self-sovereign identity schemes, such as those compliant with the [Decentralized Identifier (DID) Data Model](https://w3c.github.io/did-core/). 

DCC is especially interested in interoperability and portability enabled by DIDs. The DID core standard enables intereoperability across DID methods, including those that are bootstrapped from existing trust sources, such as a well known web domain.

While DCC doesn't mandate use of a specific DID method, we have made choices in our initial implementations/pilots, which are described below. 

## Issuer Identifiers

The [issuer identity registry](issuer_registry.md) is the authoritative source for membership. This is the place to consult to discover, e.g. Univerity of Toronto is a DCC member, and uses a specific set of dids associated with their credentials.

For an individual issuer's did method, we've chosen to start with the [did:web method](https://w3c-ccg.github.io/did-method-web/). This is largely a matter of trust and convenience in our early implementations; our members have long-standing web domains and established processes for pushing updates. So, for example the University of Toronto would host a DID document per the did:web method, and be responsible for maintaining updates to their key materials.

Again, issuers are free to use different DID methods, but must factor in the trustworthiness of the method they choose. (TODO: link to threat model)

## Learner Identifiers
 
The DCC wallet will allow learners to select an identity method that complies with the Verifiable Credential data model, which is simply a URI. That allows learners to choose a web address, [solid](https://solidproject.org) profile, or a [DID method of their choice](https://www.w3.org/TR/did-spec-registries/) such as uport ethereum DIDs.

The DCC wallet will provide a flexible identity plugin layer enabling different implementations. In our choices of built-in identity methods, we are using the  criteria below. For a more extensive analysis of assessing DID methods, see the [DID Rubric](https://w3c.github.io/did-rubric/) document.

- Inexpensive
- Portable; self-sovereign (decentralized)
- Standards-compliant, has multiple implementations
- Updateable years past OR issuer supports refresh

Portability options:
- redirect / forwarding address (service provider level forwarding)
- being able to update DID (document level forwarding)
- ask for a re-issue of credential that points to different DID

Additional factors/consideration:
- Near-term/long-term
    - e.g. for demo purposes we can start with did:key
- No perception of COI; vendor lockin
- Avoid restrictive licenses / royalty free
- Infrastructure requirements:
    - Who hosts: DCC, existing ledger
    - DID:web -- edu insts already do this
- Length of DID / length of credential
    - fitting into QR codes
    - Note: would be nice to fit all into QR code, but not required initially


The wallet will provide an initial implementation of [did:key](https://w3c-ccg.github.io/did-method-key/) for pilots, with the additional issuer requirement to allow reissuance in case of loss.

## Next Steps and Additional Considerations

These are our design choices for the initial pilots, and are subject to revision over time as we evaluate robustness and capabilities of different DID methods. 

Associated with our design chocies, we are developing threat models for implementors/vendors to factor into their solutions and workflows.

Future versions will provide clearer guidance for management of multiple issuer DIDs. This concern is as follows (per university):
- A registrar will have a DID (or set of DIDs) for diplomas and transcripts
- Other credential issuers throughout the university will have different DIDs for a variety of use cases, such as: online courses, course completion, competency- or skill-based credentials

Specifically, a registrar may not want credentials issued for an individual course to have the same weight has official university transcripts or diplomas. This effort will be related to improvements on the Issuer Registry.


## Other DID Methods we considered 

- did:key (*now*)
    - decision: demo, possibly longer term 
       - possible problem: no rotation properties
       - but may be able to use longer term with reissuance
    - Implementations exist; can do raw javascript
        - later on, can link to native
- sidetree (*incubate*)
    - long form
    - still considering
    - problems:
        - long form didn't work
        - spec process
- did:web (*incubate; use-case specific*)
    - need to design structure (dcc vs per uni)
    - concerns: 
        - requirements for issuers
        - portability
    - options:
        - host something: directory or subdomain; different reputation implications
        - DCC host vs university host
- did:peer
    - decision: not suitable for cred subject
- DID method based on keys.pub
    - about
        - evolution of keybase
    - conclusion
        - look at it for tech and to inform our specs
        - at the moment, operates on individual keys
        - don't have concept of multiple devices (like keybase did)
- IPFS-based
    - DCC or someone would have to run IPFS network (in between blockchain and did:web)
    - maybe for pilot 2

### Details

#### did:web for learners

Using [did:web](https://w3c-ccg.github.io/did-method-web/) for learners could be an appealing option if the issuer (or some service provider) is able to guarantee ongoing availability of the service to learners, but it introduces an ongoing dependency on the issuer. If there were clearer guidance on how to transfer DIDs cross-ledgers, this might be increasingly compelling as a way to bootstrap learner DIDs. Ultimately, we decided this introduced too much dependency on the issuer, which was counter to the decentralized design goal.

#### did:sidetree

The [sidetree did method](https://identity.foundation/sidetree/spec/) was appealing in that it has a long-form variant (generative, similar to did:key) that can then be upgraded to an on-chain sidetree DID, supporting lifecycle management options such as key rotation. At the time we investigated, this method wasn't sufficiently stable (in spec or library support for the shortform version). We will revisit in the future.

#### DNS-based DIDs

The [DNS-based DID approach](https://tools.ietf.org/html/draft-mayrhofer-did-dns-01) is similar to did:web in that it takes advantage of already well-known domain names, and is similarly an effective way to bootstrap a DID for an already-trusted entity. It could be considered to have a slight security advantage over did:web, in that the threat model doesn't need to include the web server (serving the DID document). However, the management usability was seen as prohibitive.

## Follow-up Research

- Portability of DID methods
- Compact off-chain methods for learners (new)
- Cooperative learner management (e.g. if college or funding disappears)
    - FDIC-type insurance
    - Vouching for past issuance
    - Issuer key registry
