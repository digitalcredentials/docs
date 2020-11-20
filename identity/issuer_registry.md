# Issuer Registry MVP

## Design Goals

The issuer registry stores identifiers (Decentralized Identifiers) for member institutions. It functions as a web-of-trust endorsement among the members of the consortium on each others' root of trust identifiers. 

In this (initial) implementation:
- verifiers interact with it as a simple inclusion check in the DCC registry, consisting of DCC members
- members have the option of which entities to store in the registry -- it could be 1 per institution, or multiple (i.e. one for the registrar, one for a department, etc)
- each issuer (root-of-trust entity) takes responsibility for manages their keys, key management policy, relationships with other issuing entities, capabilities, etc
- DCC will provide recommendations, best practices, and open source libraries to help implement

The goal for the MVP / first iteration, is  to put something in place that performs the useful function of an issuer registry / directory, has lightweight governance and data model, and starts the larger conversation about the role that DCC and other organization and consortia can play.

This provides a lightweight quality control measure, manifesting during verification and functions like the twitter "blue-check" mark that identity verification has occured, addressing the concern with other decentralized solutions that there is little trust in the identity and quality. Tying the issuer identifier to well-known web domains is one part, but not the complete picture -- i.e. it still would be easy to socially engineer trust in a credential that _looks_ like the name of a well-known issuer. 

At the same time, this approach can be duplicated by other entities/trust groups.


## Overview of Approach

The barebones issuer registry is essentially a dictionary, mapping DIDs of recognized/participating institutions to a data structure that contains, at minimum:

* Institution name (string)
* Location (freeform string, its main purpose is name disambiguation, which is [a tough problem](https://www.researchgate.net/publication/277522506_Institution_name_disambiguation_for_research_assessment) in general, but we don't have to solve it 100%.)
* URL (link to its website)

This can be implemented as a JSON object, unsigned at first, although at later stages, it might be placed inside Verifiable Credential.

## Example Registry

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

## Location and Governance

To start with, the registry is hosted on GitHub Pages, in the `digitalcredentials` org: `https://digitalcredentials.github.io/issuer-registry/registry.json`

The registry will be populated with an initial list of participating universities (that are in the DCC TWG). 
To be added to the list, an institution has to do the following two things:

1. Make a PR to the `issuer-registry` repo, and add an entry for their institution.
2. Email the project requesting to be added.

The second step is optional, but can provide just enough structure / verification to be useful, but not be too gate-keeping or restrictive (meaning, we can check that it's from an .edu email address, verify with the consortium contact, etc).

## Additional Implementation Considerations

1. OK for this registry to have multiple DIDs from the same university
2. Think about stability of contract between verification and registry
3. Consider adding a check to the `sign-and-verify` fastify app so that on startup, it checks to make sure that its issuer DID (from its config) is present in the DCC `issuer-registry`, and if not, give a warning, with a link to instructions on how to add it.
    - sign-and-verify should trust itself
4. Modify the verify logic of `sign-and-verify` to fetch the registry (with caching), and check to make sure the VC issuer's DID is present there. (We can make this an optional parameter, so that people can experiment with the app before adding their institution to the registry.)

## Future Implementation and Design Considerations

- registry is unsigned for now, but later can be signed/VC
- what sort of DID and keys should DCC itself have
- website-hosted for now, but good candidate for blockchain implementation for auditability
- longer term: can also compare against issuer-hosted lists

