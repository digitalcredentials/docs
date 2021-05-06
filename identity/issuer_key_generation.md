# Issuer Key Generation

We currently recommend and support only ED25519 issuing keys. You can generate an ED25519 key in any manner you choose. The following steps describe a way to generate issuing keys as part of pre-configured DID documents, which can be used with the DCC sign-and-verify libraries.

The sign-and-verify service and sign-and-verify-core library expect pre-configured Ed25519 keys to be encoded as specified in [Ed25519VerificationKey2020](https://github.com/digitalbazaar/ed25519-verification-key-2020). These can be loaded as part of DID documents of 2 styles: public and "unlocked". 

- The verifier endpoints/functions only need the public variant
- The issuing/signing endpoints/functiond require the "unlocked" versions, which should be stored securely.

The [didgen](https://github.com/digitalcredentials/didgen) utility helps you generate public and unlocked DID documents for signing and verifying. Usage information is available in the README.

Note that the unlocked versions contine the `privateKeyMultibase` component:


```
...
{
  type: 'Ed25519VerificationKey2020',
  id: 'did:example:1234#z6MkszZtxCmA2Ce4vUV132PCuLQmwnaDD5mw2L23fGNnsiX3',
  controller: 'did:example:1234',
  publicKeyMultibase: 'zEYJrMxWigf9boyeJMTRN4Ern8DJMoCXaLK77pzQmxVjf',
  privateKeyMultibase: 'z4E7Q4neNHwv3pXUNzUjzc6TTYspqn9Aw6vakpRKpbVrCzwKWD4hQDHnxuhfrTaMjnR8BTp9NeUvJiwJoSUM6xHAZ'
}
...
```

The `privateKeyMultibase` indicates this is a secret, which should never be shared. Store the results of didgen as described in the usage information; you will need this for signing the VC and registering the issuer identity. 

