# Learner Identity

DCC uses [Decentralized Identifiers](https://w3c.github.io/did-core/) to support learner identity. Learners may select any Decentralized Identifier method.

Our credential wallet implementation will support flexible DID providers, with default option(s) that are economical, trustworthy, and prevent lockin. 

We are determining initial implementations for the learner wallet. Design discussion follows

## Criteria

- Inexpensive
- Portable; self-sovereign (decentralized)
- Standards-compliant, has multiple implementations
- Updateable years past OR do we need to update it

Portability options:
1. redirect / forwarding address (service provider level forwarding)
2. being able to update DID (document level forwarding)
3. ask for a re-issue of credential that points to different DID 

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

## Methods we considered 

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
 

## Follow-up Research

- Portability of DID methods
- Compact off-chain methods for learners (new)
- Cooperative learner management (e.g. if college or funding disappears)
    - FDIC-type insurance
    - Vouching for past issuance
    - Issuer key registry
