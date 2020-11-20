# Associating a signing key with a demo issuer identity

The [did:web decentralized identifier (DID) method](https://w3c-ccg.github.io/did-method-web/) provides a simple way to start experimenting with VC/DID standards. This method allows bootstrapping trust using a web domain’s existing reputation.

Many aspects of the DCC open standard works with any DID method, but verifiers/relying parties may not accept all DID methods. You should be very careful to consider the threat model of the method you choose.


## Create/update the DID document

First create or update the DID document. DCC uses the [JSON Web Signature 2020 Linked Data key descriptions](https://w3c-ccg.github.io/ld-cryptosuite-registry/#jsonwebsignature2020).

The following is an example of the DID document you will create. Note the following:
- `id` corresponds to the DID; `did:web:<domain>`
- Note that `publicKey.id` is `did:web:<domain>`#`kid`
- `publicKeyJwk` contains the key generated in the previous step, omitting `d`
- By default, we've granted our key authentation, assertion, capability delegation, and capability invocation abilities.

```
{
  "@context": "https://w3id.org/did/v0.11",
  "id": "did:web:digitalcredentials.github.io",
  "publicKey": [
    {
      "id": "did:web:digitalcredentials.github.io#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs",
      "type": "JwsVerificationKey2020",
      "controller": "did:web:digitalcredentials.github.io",
      "publicKeyJwk": {
        "kid": "96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs",
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "NCqHLgxwYX0GJO2phSUBHZ-w0Tr5sblr7bCZHZ2ld_I"
      }
    }
  ],
  "authentication": [
    "#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs"
  ],
  "assertionMethod": [
    "#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs"
  ],
  "capabilityDelegation": [
    "#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs"
  ],
  "capabilityInvocation": [
    "#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs"
  ]
}
```

If you already have a DID document, just add your new public key to the `publicKey` array, and the ids to each "ability" array 

## Serve the DID document at the appropriate location

Per the did:web method, add/update the DID document `<domain>/.well-known/did.json`.

You can see an example of the [DCC samples DID document](https://digitalcredentials.github.io/samples/.well-known/did.json).

