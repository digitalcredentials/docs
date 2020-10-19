# Credential Request Flow

This is a representative e2e demonstration of how an issuer can integrate DCC credential requests into their systems. This demonstrates initiating the exchange with a link that performs oauth, but this can be adapted to other flows.

The only requirements are those assumed by the DCC client app interface (corresponding to the lower half of the diagram, marked "core"), including"
- DEEP_LINK to open DCC client app to the credential request page
- Server "request credential" endpoint to accept DCC client app's request

![](cred_request_cropped.png)

## ORIGINAL_LINK

`ORIGINAL_LINK` contains the following information needed by the issuer and the DCC client app:
- Authentication URL
- Credential Request URL
- Deep link (platform-specific)
- Challenge (for the DCC client digital signature)
- Any other state needed by the issuer app

With oauth, the `ORIGINAL_LINK` sent to the user would have the following structure:

```
https://<issuer_authentication_url>/authorize
   ?client_id=<client_id>        // provided by issuer
   &response_type=code
   &state=<state>                // DCC client state; not used for mobile scenarios
   &redirect_uri=<redirect_uri>  // deep link into DCC client with request params (or intermediate service)
   &scope=<scope>                // if needed by issuer
```

`redirect_uri` is either a deep link into the DCC client app or an intermediate service that provides that link. It's described in detail in the next section

## Request Credential Initial State

After successful authentication, the user is redirected (via `redirect_uri` or intermediate service) into the DCC Wallet "Request Credential" screen. 

For the DCC mobile app client, `redirect_url` looks like:

```
dcc:request?                         // mobile app deep link
    request_url=<request_url>        // credential request url, encoded
    &challenge=<challenge>           // challege for signing
```

We'll refer to this as `DEEP_LINK`. 

Details:
- The `request_url` value is required by the DCC client app. That is the issuer's credential request endpoint the DCC client app will hit
    - May include additional parameters that the DCC client app will pass along. This is useful, for example, for specifying additional state needed to lookup the subject's credential to be issued 
    - e.g. [&issuance_id=<issuance_id>...]
- The `challenge` is required by the DCC client app for inclusion in the signed message sent to the request URL

The wallet generates the `did`.

## Build Credential Request

Note: Wallet uses `sign-and-verify` library to create and sign the VP in the following steps.

- Wallet creates a Verifiable Presentation with the following information
  - `did`
  - `challenge`
  - `presentation_id` (optional)
- Signs the VP
- On click, sends signed VP to `request_url`

## Build and Sign Credential

This details the final step of the of the Credential Request ceremony. (see "Issuer Credential Request Endpoint" in the diagram above)

This is the flow:
1. Check auth state, as needed
2. Verify the subject's `did`
    - Use `sign-and-verify` library/service
    - Endpoint: `/verify/presentations`
    - Details below
3. If verified, extract (and store) the subject's DID
4. Lookup/construct credential
    - Use session state to lookup which credential we want to issue to the subject, and construct credential
    - Add the subject DID (from step 3) to the credential (`credentialSubject.id`)
5. Sign (issue) the credential 
    - Use `sign-and-verify` library/service
    - Endpoint: `/issue/credentials`
6. Store credential and related accounting information
7. Return the VC


### Step 2 Details: Verify VP

The VP verification payload contains the following:
- Verifiable Presentation provided from subject's wallet (containing `did` + `challenge`, signed)
  - pass-through
- Options, constructed by issuer

The outer structure looks like this:
```
curl --header "Content-Type: application/json"  \
    --request POST --data '<VP verification payload>'  \
    <sign-and-verify-endpoint>/verify/presentations
```

"VP verification payload" looks like this:

```
{
  verifiablePresentation: "<payload from subject>",
  options: {
    verificationMethod: "<Parsed from learner's VP>",
    challenge: "<Expected 1-time challenge>"
  }
}
```

Note: the `/verify/presentations` API.contract comes from [vc-http-api](https://w3c-ccg.github.io/vc-http-api/). It's awkward, because `options`, constructed by issuer, always seems to use the  `verificationMethod` provided from the subject's request. I have a tracking issue to clarify, and we can change default behavior on sign-and-verify to recognize this.


### Example

Assumptions:
- sign-and-verify is running locally on port 5000
- subject DID is `did:web:digitalcredentials.github.io` in this example. This is atypical because I reused issuer DID (I'll update later; I just wanted to include a payload that I verified)

Experimental code (partially) demonstrating this: https://github.com/digitalcredentials/issuer-demo

#### CURL command to verify VP

```
curl --header "Content-Type: application/json" --request POST --data '{"verifiablePresentation": {"@context":["https://www.w3.org/2018/credentials/v1"],"type":["VerifiablePresentation"],"id":"456","holder":"did:web:digitalcredentials.github.io","proof":{"type":"/JsonWebSignature2020","http://purl.org/dc/terms/created":{"type":"http://www.w3.org/2001/XMLSchema#dateTime","@value":"2020-10-12T17:06:49.767Z"},"https://w3id.org/security#challenge":"123","https://w3id.org/security#jws":"eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..4OiWb5EGPmXhtMNhmVXwyYhUI2BLbgcP0o-GNQaXBsMARfEGMTZi28BDiXmkdsCWvx2xmFD-cROvyIr-qMpeCQ","https://w3id.org/security#proofPurpose":{"id":"https://w3id.org/security#authenticationMethod"},"https://w3id.org/security#verificationMethod":{"id":"did:web:digitalcredentials.github.io#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs"}}}, "options": {"verificationMethod": "did:web:digitalcredentials.github.io#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs", "challenge":"123"}}' http://127.0.0.1:5000/verify/presentations
```


#### Verifiable Presentation (formatted):
Formatted for clarity. This payload is passed through from subject:
```
{
    "@context": [
      "https://www.w3.org/2018/credentials/v1"
    ],
    "type": [
      "VerifiablePresentation"
    ],
    "id": "456",
    "holder": "did:web:digitalcredentials.github.io",
    "proof": {
      "type": "/JsonWebSignature2020",
      "created": {
        "type": "http://www.w3.org/2001/XMLSchema#dateTime",
        "@value": "2020-10-12T17:06:49.767Z"
      },
      "https://w3id.org/security#challenge": "123",
      "https://w3id.org/security#jws": "eyJhbGciOiJFZERTQSIsImI2NCI6ZmFsc2UsImNyaXQiOlsiYjY0Il19..4OiWb5EGPmXhtMNhmVXwyYhUI2BLbgcP0o-GNQaXBsMARfEGMTZi28BDiXmkdsCWvx2xmFD-cROvyIr-qMpeCQ",
      "https://w3id.org/security#proofPurpose": {
        "id": "https://w3id.org/security#authenticationMethod"
      },
      "https://w3id.org/security#verificationMethod": {
        "id": "did:web:digitalcredentials.github.io#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs"
      }
    }
}
```

#### Options (formatted):

Formatted for clarity. 

Notes:
- `verificationMethod` has the awkwardness I described above
- issuer should provide `challenge` to ensure the value in the VP payload is what we expect

```
{
  "verificationMethod": "did:web:digitalcredentials.github.io#96K4BSIWAkhcclKssb8yTWMQSz4QzPWBy-JsAFlwoIs",
  "challenge": "123"
}
```
