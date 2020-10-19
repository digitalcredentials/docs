# Verify Credential

Upon receipt of the learner's credential, the relying party checks via a DCC library (TODO: link) or service:

1. The issuer’s identifier can be found in the DCC registry (smart contract)
2. The issuer's signing key (in the credential) is valid and corresponds to the value dereferencable from the DCC registry
3. The digital signature indicates the credential has not been tampered with and that the issuer's signing key was in fact used to sign.
4. The credential has not expired, according to the expiration terms provided by the issuer in
the credential.
5. The credential’s status has not changed to an unusable state, per check of the status registry embedded in the credential.
