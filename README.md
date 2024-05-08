# docs

## Table of Contents

- [About DCC](#about-dcc)
- [Overview of Components](#overview-of-components)
  * [Credentials](#credentials)
  * [Identity](#identity)
  * [Credential Issuing and State Management](#credential-issuing-and-state-management)
  * [Verifying](#verifying)
  * [Wallet and Storage](#wallet-and-storage)
- [Design, requirements, threat model](#design--requirements--threat-model)
- [Repositories](#repositories)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## About DCC

The [Digital Credentials Consortium](https://digitalcredentials.mit.edu/) was founded by leading universities with expertise in the design of verifiable digital credentials. Together, we are designing an infrastructure for digital credentials of academic achievement.

We are committed to open standards and encourage all to contribute through the [W3C VC-EDU task force](https://w3c-ccg.github.io/vc-ed/). For more details on how to implement our technologies and for non-technical information, check out our [Knowledge Base](https://wiki.dcconsortium.org).

## Overview of Components

For additional context, the [DCC whitepaper](https://wiki.dcconsortium.org/app/page/1EN-emRW0Rc5Rrw9KcKkKBnEtEa8YqR-y?p=1h1VJHHv2zSe0n9Ltg-KQevSuqWbGcExy) outlines our approach and high-level architecture. 

Overview of the Digital Credentials Consortium components:

### Credentials

DCC uses the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/) with linked data. 
- [Authoring guide](authoring/README.md) - authoring DCC credentials
- [Modeling Educational Verifiable Credentials](https://w3c-ccg.github.io/vc-ed-models/) 
  - see similar standards incubation efforts at the [W3C VC-EDU task force](https://w3c-ccg.github.io/vc-ed/)

### Identity

- [Identity design decisions](identity/design_decisions_id.md)
- [Issuer registry MVP design](identity/issuer_registry.md)
- [Issuer Key and DID Document Generation](identity/issuer_key_generation.md)
- [Setting up an issuer issuing identity](identity/issuer_demo_id.md)

### Verified Issuance

Onboard the issuer into the [issuer registry](identity/issuer_registry.md)


### Credential Issuing and State Management

Issuing credentials and maintaining state involves:

- [Credential request flow](request/credential_request.md)
  - [Issuing a credential to the learner](credential/issue_credential.md)
- [Revoke a credential](credential/revoke_credential.md)
  - [Revocation design discussions](credential/revoke_credential_design_discussions.md)

### Verifying

Verifying credentials involves:
- [Verification of the credential](verification/verify_credential.md)
- Identity proofing, variable (coming soon)

### Wallet and Storage
DCC uses the [Verifiable Credentials Data Model](https://w3c.github.io/vc-data-model/) and can  

- [cred-wallet](https://github.com/digitalcredentials/cred-wallet): mobile wallet that manages learner's credentials and identifiers

### Design, requirements, threat model

- [system requirements](system/requirements.md)
- [threat model](system/threat_model.md)

## Repositories

- [docs](https://github.com/digitalcredentials/docs): (this repo) contains project documentation, broader architectural docs, etc
- [sign-and-verify](https://github.com/digitalcredentials/sign-and-verify): library to sign/issue and verify DCC credentials. 
- [credgen](https://github.com/digitalcredentials/credgen): command line tool to generate credential templates
- [learner-credential-wallet](https://github.com/digitalcredentials/learner-credential-wallet): repository for DCC react native mobile app to manage and store credentials 
- [designer](https://github.com/digitalcredentials/designer): web ui to design credential templates -- intended for developers

