## Phase 1 / MVP

The goal for the MVP / first iteration, should be just to put something in place that performs the useful function of an issuer registry / directory, has lightweight governance and data model, and starts the larger conversation about the role that DCC and other organization and consortia can play.

### Overview

Create a barebones issuer registry that's essentially a dictionary, mapping DIDs of recognized/participating institutions to a data structure that contains, at minimum:

* Institution name (string)
* Location (freeform string, its main purpose is name disambiguation, which is [a tough problem](https://www.researchgate.net/publication/277522506_Institution_name_disambiguation_for_research_assessment) in general, but we don't have to solve it 100%.)
* URL (link to its website)

This can be implemented as a JSON object, unsigned at first, although at later stages, we can put it inside a Verifiable Credential (once the subject of "so what sort of DID and keys should DCC itself have?" gets sorted out).

### Example Registry

```json
{
  "meta": {
     "created": "2020-12-01 ...",
     "updated": "..."
  },
  "registry": {
     "did:example:1234": {
        "name": "Example University",
        "location": "San Diego, CA, USA",
        "url": "https://www.example.edu"
     }
  }
}
```

### Location and Governance

To start with, hosted on GitHub Pages, here in the `digitalcredentials` org, e.g.:

`https://digitalcredentials.github.io/issuer-registry/registry.json`

The registry will be populated with an initial list of participating universities etc (that are in this working group). 
To be added to the list, an institution has to do the following two things:

1. Make a PR to the `issuer-registry` repo, and add an entry for their institution.
2. Email the project requesting to be added. (This can go to a mailing list or just an email address that Kim and others can check).

The second step is optional, but can provide just enough structure / verification to be useful, but not be too gate-keeping or restrictive (meaning, we can check that it's from an .edu email address, etc).

### Implementation
Couple of notes on implementation details:

1. We can add a check to the `sign-and-verify` fastify app so that on startup, it checks to make sure that its issuer DID (from its config) is present in the DCC `issuer-registry`, and if not, throw an error or give a warning, with a link to instructions on how to add it.
2. Modify the verify logic of `sign-and-verify` to fetch the registry (with caching of course), and check to make sure the VC issuer's DID is present there. (We can make this an optional parameter, so that people can experiment with the app before adding their institution to the registry.)

### Additional Clarifications

- ok for this registry to have multiple DIDs from the same university
- sign-and-verify trusts itself
- registry is unsigned for now, but later can be signed/VC. 
- website-hosted for now, but good candidate for blockchain implementation for auditability
- Think about stability of contract between verification and registry
- Longer term: can compare against issuer-hosted
