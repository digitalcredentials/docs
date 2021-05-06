# Associating a signing key with a demo issuer identity

The [did:web decentralized identifier (DID) method](https://w3c-ccg.github.io/did-method-web/) provides a simple way to start experimenting with VC/DID standards. This method allows bootstrapping trust using a web domain’s existing reputation.

Many aspects of the DCC open standard works with any DID method, but verifiers/relying parties may not accept all DID methods. You should be very careful to consider the threat model of the method you choose.


## Create/update the DID document

First create or update the DID document as described in [Issuer Key and DID Document Generation](issuer_key_generation.md)

The following is an example of the DID document you will create. Note that `id` corresponds to the DID; `did:web:<domain>`
```
{
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "id": "did:web:digitalcredentials.github.io",
  "assertionMethod": [
    {
      "id": "did:web:digitalcredentials.github.io#z6MkrXSQTybtqyMasfSxeRBJxDvDUGqb7mt9fFVXkVn6xTG7",
      "type": "Ed25519VerificationKey2020",
      "controller": "did:web:digitalcredentials.github.io",
      "publicKeyMultibase": "zD5BMsjMTWRs7mAcFxrDU78NDehZjhtdnyEabvDp63EUj"
    }
  ]
}
```

If you already have a DID document, just add your new public key to the `assertionMethod` array. 

## Serve the DID document at the appropriate location

Per the did:web method, add/update the DID document `<domain>/.well-known/did.json`.

You can see an example of the [DCC samples DID document](https://digitalcredentials.github.io/samples/.well-known/did.json).

