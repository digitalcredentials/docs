

## MVP Revocation

We believe that for the DCC MVP/Pilot project, selecting the [Revocation List  2020 bitstring strategy](https://w3c-ccg.github.io/vc-status-rl-2020/) strikes the right balance of implementation simplicity (leveraging [an existing library](https://github.com/digitalbazaar/vc-revocation-list/)), reasonable server-side storage constraints, cache-ability and herd privacy, for the given use case.

### First Iteration - Randomly Named Block with Sequential Index Assignment

For the initial implementation:

1. Use the [RevList 2020 library](https://github.com/digitalbazaar/vc-revocation-list/) with the default bitstring block size of 16KB (131k entries).
2. Store the (zlib-compressed, per the spec) revocation list on disk (so that it can be served as a static file by the issuer app). Use a random string as the filename (for example, `QHDR12CD2J`, so that it can be served at a URL like `https://example.edu/revocations/QHDR12CD2J`).
3. The issuer app will keep track of the "number of credentials issued" count, internally. Whenever a new VC is issued, that count will be incremented, and used as the position of the revocation index.
4. Once the "number of credentials issued" count goes past 131k entries, a new bitstring block is generated (with a new random file name), and the count is reset.

#### Implementation Note - Revocation Config

The issuer service will create a `revocation.config.json` file on startup, and will use it to keep track of the `credentialsIssued` count, as well as the `currentBlock` filename.
So, for example, the first time an issuer service is started, it will check to see if the `revocation.config.json` file exists. If not, it will generate a new one. For example:

```json
{
  "credentialsIssued": 0,
  "currentBlock": "QHDR12CD2J"
}
```

That means the first VC issued by that issuer service will have the following `credentialStatus` attribute:

```js
{
  // "@context": [ ... ],
  "credentialStatus": {
    "type": "RevocationList2020Status",
    "revocationListIndex": "1",
    "revocationListCredential": "https://example.edu/revocations/QHDR12CD2J"
  },
  "credentialSubject": { /* ... */ },
  "proof":{ /* ... */ }
}
```

The next VC issued will have the `revocationListIndex: "2"`, and so on.

#### Implementation Note - Issuer Log

To aid the revocation process, the Issuer service should keep a private log of issued VCs. For each VC, the log will note:

* The credential `id`, and any other relevant metadata (timestamp, subject id, which key it was issued with, etc).
* The `revocationListCredential` url (or simply the bitstring filename) that was used.
* The `revocationListIndex` for that credential.

That way, when a particular VC needs to be revoked, the issuer log will contain the filename and the index position for that credential, so that its bit can be flipped.

#### First Iteration - Limitations/Risks

1. Revocation list is centralized at the issuer server; issuer can track verifier requests. However, see the section on Cache Expiration below, for mitigation.
2. For a given revocation bitstring URL, an attacker can estimate the number of credentials that were revoked (just by counting the revoked bits), which could give them an approximate percentage of VC revocation rates (for that block). (This can be mitigated by initializing the bitstring with noise, see below.)
3. Because the indexes are sequential, an attacker that sees multiple VCs can estimate the number of total VCs issued _for that block_. See the [German Tank Problem](https://en.wikipedia.org/wiki/German_tank_problem) for details. Note that because the revocation list is randomly named, the attacker does not necessarily know how many other revocation list blocks exist (and therefore, what the total number of VCs issued is).

### Second Iteration - Randomly Named Block with Random Index Assignment

To mitigate risk 3 mentioned above (the ability of an attacker to estimate the number of VCs issued for a given block), a second iteration can be done.

Instead of assigning list indexes sequentially, the issuer will generate the list index randomly. Because of the random assignment, the issuer will need a _second_ bitstring, this one private, to keep track of which indexes have already been assigned.

This approach doubles the server-side storage requirements (for 131k entries, now 32KB is required, versus the 16KB in the first iteration). However, note that there is no additional bandwidth or storage burden on the verifier / relying party.

### Optional Enhancement - Block Initialization Using Noise/Chaff

To mitigate risk 2 (ability of an attacker to estimate the percentage of the VCs revoked for a given block), an additional step can be taken.

When a revocation bitstring block is provisioned by the issuer, a percentage of it (say, 20-30%) can be initialized with random noise (so, 30% of the bits can be randomly flipped to 1, at the outset). To keep track of which index positions have been chaffed in this manner, they can be recorded in the second, private bitstring from the Second Iteration above.

Note that this does increase the issuer-side storage requirement by the same percentage (ie, by 20-30% from the previous example).

### Cache Expiration and Mitigating Tracking/"Phone Home"

The major drawback of a bitstring revocation list stored at the issuer's website is the possibility of the issuer being able to track the verifier's requests for a given revocation list. This can potentially provide metadata such as timestamp, the requester's IP address, and any cookies and headers the verifier's browser is willing to send.

However, depending on the use case (the relative value and threat level of the credential), several simple steps can be taken to mitigate this risk.

Firstly, not all VC use cases _need_ to be revokable. For each type of credential, the issuer is advised to consider whether their business processes actually allow for revocation. In many cases, the issuer can use a VC expiration date instead of revocation (that is, issue a transcript that is good for one year, for example).

For VC use cases that do need revocation ability, issuers can reduce the "phone home" problem by considering just _how quickly_ does a particular VC need to be revoked. This is an important question, because it directly determines the cache-ability of the revocation list, and whether it can be stored by third parties (for example, [cached by Content Delivery Networks](https://www.cloudflare.com/learning/cdn/glossary/what-is-cache-control/)).

If a high-value/high-threat credential (such as an administrator login) needs to be be revoked (and the revocation needs to propagate) _within a matter of seconds_, it is not very cache-able, and the issuer's revocation list must be the only source of truth.

If, however, the business rules require that the revocation can propagate within a day, that means that the revocation list can be cached by a CDN (or by the verifier's browser cache) with the cache expiration value of 24 hrs.

This dramatically changes the risk of tracking / "phone home" behavior by the issuer. Now, instead of a verifier having to request the revocation list _each time_ it verifies a VC, each verifier can simply fetch the revocation list first thing in the morning. This can be done automatically (by a cron job, etc, much like RSS feeds), which means that all subsequent verification operations (within that 24 hr expiration period) can be performed using the verifier's local cache. Well known HTTP mechanisms (such as the `Etag` / [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match) headers) allow further bandwidth savings for verifiers and issuers alike.
