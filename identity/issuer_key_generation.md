# Issuer Key Generation

We currently recommend and support only ED25519 issuing keys. You can generate an ED25519 key in any manner you choose. 

An example of a tool you can use is [jwkgen](https://github.com/rakutentech/jwkgen), which will output a key in the following format. 

```
{
  "kid": "",
  "kty": "OKP",
  "crv": "Ed25519",
  "x": "",
  "d": ""
}
```

Save the results of this to a secure location; you will need this for signing the VC and registering the issuer identity. The `d` component indicates this is a private key, which should never be shared.


