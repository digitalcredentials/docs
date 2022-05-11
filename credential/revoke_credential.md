

## MVP Revocation

We believe that for the DCC MVP/Pilot project, selecting the [Status List 2021 bitstring strategy](https://w3c-ccg.github.io/vc-status-list-2021) strikes the right balance of implementation simplicity (leveraging [an existing library](https://github.com/digitalbazaar/vc-status-list)), reasonable server-side storage constraints, cache-ability and herd privacy, for the given use case.

### First Iteration - Randomly Named Block with Sequential Index Assignment

#### Status Config

The issuer service will create a `config.json` file in a dedicated credential status folder (`CRED_STATUS_FOLDER_DISK`) on startup, and will use it to keep track of the number of credentials issued (`credentialsIssued`), as well as the unique ID of the latest status block (`latestBlock`).
The first time an issuer service is started, it will check to see if the `config.json` file exists. If not, it will generate a new one. For example:

```json
{
  "credentialsIssued": 0,
  "latestBlock": "QHDR12CD2J"
}
```

That means the first VC issued by that issuer service will look something like the example below:

```js
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/vc/status-list/2021/v1"
  ],
  "id": "https://example.edu/credentials/12784910835",
  "type": ["VerifiableCredential"],
  "issuer": "did:example:12345",
  "issued": "2021-04-05T14:27:42Z",
  "credentialStatus": {
    "type": "StatusList2021Entry",
    "statusPurpose": "revocation",
    "statusListIndex": "1",
    "statusListCredential": "https://example.edu/status/QHDR12CD2J"
  },
  "credentialSubject": { /* ... */ },
  "proof":{ /* ... */ }
}
```

Here are the most important properties to note in the `credentialStatus` property:

- `statusListIndex`: position in the bitstring that represents the status of this credential (in our implementation, this value increments by 1, so that the next VC issued will have a value of `2`
- `statusListCredential`: location of status credential (defined below)

#### Status Credential

We will manage the status of credentials with a `StatusList2021Credential`, as defined in the [Status List 2021 spec](https://w3c-ccg.github.io/vc-status-list-2021). We provide an example below:

```js
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://w3id.org/vc/status-list/2021/v1"
  ],
  "id": "https://example.edu/status/QHDR12CD2J",
  "type": ["VerifiableCredential", "StatusList2021Credential"],
  "issuer": "did:example:12345",
  "issued": "2021-04-05T14:27:40Z",
  "credentialSubject": {
    "id": "https://example.edu/status/QHDR12CD2J#list",
    "type": "StatusList2021",
    "statusPurpose": "revocation",
    "encodedList": "H4sIAAAAAAAAA-3BMQEAAADCoPVPbQwfoAAAAAAAAAAAAAAAAAAAAIC3AYbSVKsAQAAA"
  },
  "proof": { ... }
}
```

Here are the most important properties to note:

- `id`: same value referenced in the `credentialStatus.statusListCredential` path of the main credential
- `type`: indicates that this is a verifiable credential that conforms to the Status List 2021 data model
- `credentialSubject.statusPurpose`: indicates the class of status that is tracked by this credential
- `credentialSubject.encodedList`: tracks status of several credentials as a bitstring, where each bit represents the status of a credential

#### Issuer Log

To aid the revocation process, the Issuer service should keep a private `log.json` file in `CRED_STATUS_FOLDER_DISK` tracking issued VCs. For each VC, the log will note:

* The credential `id`, and any other relevant metadata (timestamp, subject id/email, action (e.g., `revoked`), which key it was issued with, etc)
* The `statusListCredential` url (or simply the status block ID) that was used
* The `statusListIndex` for that credential

That way, when a particular VC needs to be revoked, the issuer log will contain the filename and the index position for that credential, so that its bit can be flipped.

#### Issuance

1. Use the [Status List 2021 library](https://github.com/digitalbazaar/vc-status-list) with the default bitstring block size of 16KB (131k entries).
2. Store the (zlib-compressed, per the spec) status list in the `credentialSubject.encodedList` path of a `StatusList2021Credential` on disk (so that it can be served as a static file by the issuer app). Use a random string as the filename (for example, `QHDR12CD2J`, so that it can be served at a URL (`CRED_STATUS_FOLDER_URL`) like `https://example.edu/status/QHDR12CD2J`).
3. The issuer app will keep track of the number of credentials issued, internally. Whenever a new VC is issued, that count will be incremented, and used as the position of the status index.
4. Once the number of credentials issued count exceeds 131k entries, a new status block is generated (with a new random file name), and the count is reset.
5. The issuer adds an entry to `log.json` with the `issued` action.

#### Verification

1. The verifier fetches a credential's status credential at the `credentialStatus.statusListCredential` path of the main credential.
2. The verifier retrieves the status bitstring at the `credentialSubject.encodedList` path of the status credential.
3. The verifier checks the bit of the status bitstring located at the `credentialStatus.statusListIndex` path of the main credential.
4. If the value of this bit is `1`, the credential is revoked. If the value of this bit is `0`, the credential is active.

#### Revocation

1. The issuer identifies that a particular credential needs to be revoked.
2. Aided by `log.json` and other internal data, the issuer loads the status file tracking the affected credential, flips the bit at the appropriate index, and saves the file.
3. The issuer adds an entry to `log.json` with the `revoked` action.

#### Limitations/Risks

1. Status list is centralized at the issuer server; issuer can track verifier requests. However, see the section on Cache Expiration below, for mitigation.
2. For a given bitstring URL, an attacker can estimate the number of credentials that were revoked (just by counting the revoked bits), which could give them an approximate percentage of VC revocation rates (for that block). (This can be mitigated by initializing the bitstring with noise, see below.)
3. Because the indexes are sequential, an attacker that sees multiple VCs can estimate the number of total VCs issued _for that block_. See the [German Tank Problem](https://en.wikipedia.org/wiki/German_tank_problem) for details. Note that because the status list is randomly named, the attacker does not necessarily know how many other status list blocks exist (and therefore, what the total number of VCs issued is).

### Second Iteration - Randomly Named Block with Random Index Assignment

To mitigate risk 3 mentioned above (the ability of an attacker to estimate the number of VCs issued for a given block), a second iteration can be done.

Instead of assigning list indexes sequentially, the issuer will generate the list index randomly. Because of the random assignment, the issuer will need a _second_ bitstring, this one private, to keep track of which indexes have already been assigned.

This approach doubles the server-side storage requirements (for 131k entries, now 32KB is required, versus the 16KB in the first iteration). However, note that there is no additional bandwidth or storage burden on the verifier / relying party.

### Block Initialization Using Noise/Chaff

To mitigate risk 2 (ability of an attacker to estimate the percentage of the VCs revoked for a given block), an additional step can be taken.

When a status bitstring block is provisioned by the issuer, a percentage of it (say, 20-30%) can be initialized with random noise (so, 30% of the bits can be randomly flipped to 1, at the outset). To keep track of which index positions have been chaffed in this manner, they can be recorded in the second, private bitstring from the Second Iteration above.

Note that this does increase the issuer-side storage requirement by the same percentage (ie, by 20-30% from the previous example).

### Cache Expiration and Mitigating Tracking/"Phone Home"

The major drawback of a bitstring status list stored at the issuer's website is the possibility of the issuer being able to track the verifier's requests for a given status list. This can potentially provide metadata such as timestamp, the requester's IP address, and any cookies and headers the verifier's browser is willing to send.

However, depending on the use case (the relative value and threat level of the credential), several simple steps can be taken to mitigate this risk.

Firstly, not all VC use cases _need_ to be revokable. For each type of credential, the issuer is advised to consider whether their business processes actually allow for revocation. In many cases, the issuer can use a VC expiration date instead of revocation (that is, issue a transcript that is good for one year, for example).

For VC use cases that do need revocation ability, issuers can reduce the "phone home" problem by considering just _how quickly_ does a particular VC need to be revoked. This is an important question, because it directly determines the cache-ability of the status list, and whether it can be stored by third parties (for example, [cached by Content Delivery Networks](https://www.cloudflare.com/learning/cdn/glossary/what-is-cache-control/)).

If a high-value/high-threat credential (such as an administrator login) needs to be be revoked (and the revocation needs to propagate) _within a matter of seconds_, it is not very cache-able, and the issuer's status list must be the only source of truth.

If, however, the business rules require that the revocation can propagate within a day, that means that the status list can be cached by a CDN (or by the verifier's browser cache) with the cache expiration value of 24 hrs.

This dramatically changes the risk of tracking / "phone home" behavior by the issuer. Now, instead of a verifier having to request the status list _each time_ it verifies a VC, each verifier can simply fetch the status list first thing in the morning. This can be done automatically (by a cron job, etc, much like RSS feeds), which means that all subsequent verification operations (within that 24 hr expiration period) can be performed using the verifier's local cache. Well known HTTP mechanisms (such as the `Etag` / [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match) headers) allow further bandwidth savings for verifiers and issuers alike.
