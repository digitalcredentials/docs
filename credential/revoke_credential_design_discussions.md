# Revoke Credential

## Design Approach

1. For MVP / Phase 1, we implement the bit vector / [Status List 2021](https://w3c-ccg.github.io/vc-status-list-2021/) mechanism, for revocation.
2. We state that the revocationListIndex MUST be present in a valid DCC credential.

## Past Design Discussion

All Approaches Considered

- Bit vector revocation list (selected)
  - About
    - [CCG draft spec](https://w3c-ccg.github.io/vc-status-rl-2020/)
    - Index management: Issuer has list of credential to index correspondence
    - [https://lists.w3.org/Archives/Public/public-credentials/2020May/0006.html](https://lists.w3.org/Archives/Public/public-credentials/2020May/0006.html)
  - Pros:
    - Prevents phonehome and Herd privacy
      - Prevents issuer from knowing the credential was checked
      - Knows group was requested, but don’t know what
    - Compact
  - Downsides/risks:
    - need to acquire index to embed in cred before issuing
    - need to deal with sequential privacy issues; in general, be careful about assignment of indices      
  - Additional Notes:
    - Doesn’t have to be tied to bulk issuance
    - Index could be used as for "credential status", which is additional protection against fraud (every cred can be mapped to an index)
- Credential hash
  - Pros: 
    - straightforward, don't need to add info into the credential to achieve
  - Downsides / risks
    - Correlation
    - Uncertain GDPR compliance (if blockchain anchored)
    - Should avoid blockchain implementation of this (even function of PII anchored to public blockchain)
- Credential UID
  - Pros
    - OB standard
    - Easy
  - Downsides / risks
    - Centralization
    - Size
    - Information disclosure risk
    - Validation failure
  - Concern: 
    - other forms of correlation
    - issuer knows when status was requested
- Cryptographic accumulator
  - Concern: complexity 
- Expiration
