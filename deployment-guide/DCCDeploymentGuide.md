# Digital Credentials Consortium Deployment Guide

Setting up digital credential issuing typically requires integrating into existing environments with varying requirements. This guide will help you choose and configure Digital Credentials Consortium issuing components for your specific requirements.

## Table of Contents

- [Overview](#overview)
   - [Simple Signing Demo](#simple-signing-demo)
   - [Admin Dashboard Demo](#admin-dashboard-demo)
- [Definitions](#definitions)
   - [Issuer](#issuer)
   - [Holder](#holder)
   - [Verifier](#verifier)
- [Deployment Considerations](#deployment-considerations)
  - [Data Storage](#storage)
  - [Authentication](#tenants)
  - [Revocation](#revocation)
  - [Public Key Authenticity](#public-key-authenticity-registries)
  - [Private Key Security](#private-key-security)
  - [Wallets](#didweb)
  - [Verification](#verification)
  - [Credential Display](#credential-display)
- [DCC Components](#dcc-components)
  - [Overview](#services-overview)
  - [Signing Service](#sign-a-credential)
  - [Status Service](#status-service)
    - [Mongo Implemenation](#mongo-implementation)
    - [Github Implemenation](#mongo-implementation)
  - [Transaction Service](#transaction-service)
  - [Issuer Coordinator](#issuer-coordinator)
  - [Exchange Coordinator](#exchange-coordinator)
  - [Admin Dashboard](#admin-dashboard)
  - [Credential Collection Page](#credential-collection-page)
  - [Logging](#logging)
  - [Error Handling](#error-handling)
  - [Learner Credential Wallet](#learner-credential-wallet)
- [Decision Tree Guide](#decision-tree-guide)
- [Scenarios](#scenarios)
  - [Standalone System with CSV](#standalone-system-with-csv)
  - [Direct Integration](#direct-integration)
  - [Wallet Exchange - Holder Binding](#wallet-exchange)
  - [No Holder Binding](#no-holder-binding)
- [Docker Compose Examples](#docker-compose-examples)
- [License](#license)

## Overview

The [Digital Credentials Consortium](https://dcconsortium.org) provides open source software for issuing [Verifiable Credentials](https://www.w3.org/TR/vc-data-model/). Recognizing that every issuer's environment can be different, we've structured the software as a set of composable services where the services are combined and invoked by an overarching 'coordinator' service. We provide a couple of coordinators that should cover most cases, but you can also create your own to 'wire' together services according to need.  All the services and the coordinators are node express apps exposing http APIs, and are intended to run in a Docker compose network on your servers or on servers that you control. (NOTE: the DCC does not provide hosting - we strongly recommend that you run these services locally so that you control both your own data and the signing itself.)

This all sounds quite a bit more complicated than it actually is, and we've simplified deployments by publishing the services and coordinators as docker images on Docker Hub, which can be easily run with docker compose, with minimal setup and configuration.

You can in fact try different configurations of DCC services in about five minutes by installing [Docker](https://docs.docker.com/engine/install/) (if you haven't already) and then running - from a terminal - one of the following docker compose configs we've defined:

###### Simple Signing Demo

Accepts an unsigned verifiable credential, signs and returns it. Try signing a credential with two commands:

1. Start up the docker compose by pasting this into a terminal prompt and hitting return:

```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/simple-issuer-compose.yaml | docker compose -f - up```

This will download the images from Docker Hub - which might take a couple of minutes depending on your internet connection - and then starts up the issuer. You should eventually see a message like:

```
dcc-coordinator-1  | {"level":"info","message":"Server started and running on port 4005 with http","timestamp":"2024-02-17T00:26:48.567Z"}
dcc-signer-1       | Server running on port 4006
dcc-signer-1       | POST /instance/test/credentials/sign 200 60.371 ms - 1699
```

2. Once that's started up, open another terminal window and issue a credential by pasting this at the prompt and hitting return:

<details> 
<summary>Show code</summary>
	
```
curl --location 'http://localhost:4005/instance/test/credentials/issue' \
--header 'Content-Type: application/json' \
--data-raw '{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://purl.imsglobal.org/spec/ob/v3p0/context-3.0.2.json"
  ],
  "id": "urn:uuid:2fe53dc9-b2ec-4939-9b2c-0d00f6663b6c",
  "type": [
    "VerifiableCredential",
    "OpenBadgeCredential"
  ],
  "name": "DCC Test Credential",
  "issuer": {
    "type": [
      "Profile"
    ],
    "id": "did:key:z6MkhVTX9BF3NGYX6cc7jWpbNnR7cAjH8LUffabZP8Qu4ysC",
    "name": "Digital Credentials Consortium Test Issuer",
    "url": "https://dcconsortium.org",
    "image": "https://user-images.githubusercontent.com/752326/230469660-8f80d264-eccf-4edd-8e50-ea634d407778.png"
  },
  "issuanceDate": "2023-08-02T17:43:32.903Z",
  "credentialSubject": {
    "type": [
      "AchievementSubject"
    ],
    "achievement": {
      "id": "urn:uuid:bd6d9316-f7ae-4073-a1e5-2f7f5bd22922",
      "type": [
        "Achievement"
      ],
      "achievementType": "Diploma",
      "name": "Badge",
      "description": "This is a sample credential issued by the Digital Credentials Consortium to demonstrate the functionality of Verifiable Credentials for wallets and verifiers.",
      "criteria": {
        "type": "Criteria",
        "narrative": "This credential was issued to a student that demonstrated proficiency in the Python programming language that occurred from **February 17, 2023** to **June 12, 2023**."
      },
      "image": {
        "id": "https://user-images.githubusercontent.com/752326/214947713-15826a3a-b5ac-4fba-8d4a-884b60cb7157.png",
        "type": "Image"
      }
    },
    "name": "Jane Doe"
  }
}'
```

</details>

Well done you! You've cryptographically signed a credential. You can now copy the text of the credential that was returned to you, and import it into the [Learner Credential Wallet](https://lcw.app) or display/verify it at [VerifierPlus](https://verifierplus.org).

<details><summary>Explanation</summary>

There is a fair bit going on there, but some points to note:

1. The second step simply posted a preconstructed [Verifiable Credential](https://www.w3.org/TR/vc-data-model/) to the signing service, which added the signature and returned it. You can freely fiddle with the sample credential before posting it; to change the credential name, to whom it is issued, etc.
2. The credential is signed using a default signing key that we've added to our [Sandbox Registry](https://github.com/digitalcredentials/sandbox-registry). So, when verifying any credentials signed with this key, the verifier (e.g, [VerifierPlus](https://verifierplus.org) will state that the credential was signed with a test key. If you want the verifier to say that the credential was issued by you (or your institution) you'll have to generate your own key and add it to an appropriate registry. More on that in the [Public Key Authenticity](#public-key-authenticity-registries) section.
3. This particular configuration is wired together by our [issuer coordinator](https://github.com/digitalcredentials/issuer-coordinator). The 'wiring' in this case pretty much consists of simply posting the unsigned VC to the signing-service and returning the resulting signed VC. You can see that wiring [here](https://github.com/digitalcredentials/issuer-coordinator/blob/da3fab1b491d847f5dd42d43e0de1a4e8a22bba5/src/app.js#L77-L78)
4. This is probably the simplest example of using DCC services, and this might even be how you end up integrating it into your system. There are often though other factors to consider like revocation and wallets that we talk about in the [Deployment Considerations](#deployment-considerations)section.
</details>

###### Admin Dashboard Demo

This demo provides a pretty good example of all the parts of a credentialing system. You can in fact effectively run a production version of this demo as a complete standalone issuer. You may end up wanting to more directly integrate specific services into your existing systems, but this demo can give you a good sense of how things fit together.

This demo runs a docker compose with two web apps in addition to the [wallet exchanger](#exchange-coordinator) to allow:

* uploading credential data through a CSV
* creating verifiable credential templates
* creating email templates
* emailing recipients a link from which to collect a credential
* a credential collection page

```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/admin-dashboard-compose.yaml | docker compose -f - up```

Once it has started up, you can experiment with the locally running system at [http://localhost:3000](http://localhost:3000) and in particular take a look at our [Admin Dashboard Getting Started Guide](https://github.com/digitalcredentials/admin-dashboard/blob/main/docs/GETTING_STARTED.md)

The rest of this document explains factors you'll have to consider when choosing and configuring issuing components, followed by a description of the components themselves, some configurations that cover common scenarios, and a guide to setting up your own configuration.

## Definitions

Let's quickly define three fundamental terms:

### Issuer

Whoever issues the credential. A university issuing a degree for example.

### Holder

Whoever 'holds' the credential. A student whose earned a degree for example.

### Verifier

Whoever wants to verify a credential that was shared them. A job recruiter verifying a job candidate's degree for example.

## Deployment Considerations

Okay, directly into it then - there are a few fundamental factors that influence deployments:

* Data Storage
* Authentication
* Revocation
* Public Key Authenticity
* Private Key Security
* Wallets

And a couple of factors that may be outside your control, but should be considered:

* Credential Verification
* Credential Display

Let's talk about each in turn...

### Data Storage

There are two parts to this:

***Institutional Data*** - Where your institution stores your **fundamental credential data** (e.g. your student records system)

***Issuance Log*** -  Where you'll store **details about issuance** (e.g., when a credential was collected)

In some cases the two can be combined, but you might prefer not to add issuance details to your institutional system (e.g., campus PeopleSoft system) and instead - for simplicity's sake, or maybe because of institutional policy - keep those details in a separate store, like a standalone Mongo database.

Note too that in some cases you may not have an institutional store; maybe you are just issuing one-off credentials for attendance at a conference. Similarly, you may not need to log issuance. 

Let's go over these two types of data...

#### Institutional Data 

This is the fundamental authoritative data for your credentials - the [system of record (SOR)](https://en.wikipedia.org/wiki/System_of_record) for your credentials. This system may have been around for decades or even centuries (albeit in paper form), like say for the list of degrees that a university has issued.

Where your institutional data is stored, and maybe more importantly, how you can retrieve that data, will influence the structure of your issuing system. 

Let's talk about two ways you might access that data when you want to issue a credential...

##### Direct pull

If you can pull student records from the existing system 'on demand' - say when a student logs in to a credential collection page and requests something like a transcript - then you could instantly and automatically construct and return credentials on-the-fly.

A direct pull system ***might*** be more complex to setup (although often not), but the medium and long term benefits can be significant. Essentially, after having setup the system, credential holders (students/alumni) can simply download a copy of their credential whenever they like. Which also means that replacement of lost credentials is no longer a complicated issue requiring human mediated requests and lookups - the student/alumni can now simply login to get an immediate replacment, directly and automatically issued from the institutional data.  

Same goes for name changes - if someone changes their name (for example if they get married) the name change can be handled by the normal institutional process for a name change, which will end up changing the underlying institutional record. And once that is done the student/alumni then just logs back in and get a fresh copy, pulled from the update underlying data.

##### Standalone system

If you can't easily pull from your institutional store for some reason, maybe because of campus restrictions, then you may want to look at a stand-alone system to which you add credentials that you expect will be collected. 

You might add credentials to this standalone system using CSV uploads or maybe even some direct export from your institutional system.

A standalone system can be an easy way to get started with digital credentials, especially if you are initially issuing test credentials, or credentials that don't exist elsewhere, like say a certificate of attendance for a conference.

Three significant downsides to a standalone system are:

1. **Data duplication** If the credential data is already stored in some other place (e.g, campus PeopleSoft) then storing a second copy in your standalone system means that you now have two copies of the same data, which can easily get out of sync; for example, the case where someone changes their name in the main campus system, but there isn't an automatic change in your standalone system. Maintaining a second copy also introduces extra complexity and maintenance.

2. **Brittle data conversion** Adding new data to the standalone system is likely to be a manual process, which can become suprisingly complicated. If you are pulling the data out of an institutional store and then uploading it as CSV to the standalone system, you'll typically have to have various checks to confirm the data was properly pulled and added. Problems with character encoding, changed data structures, and so on can completely scuttle a project.

3. **Security** The data in your institutional store most likely has fairly rigorous security and checks to make sure that there are no:
a) data breaches where private data is stolen
b) fake data isn't added to the system.

By creating a second copy you are creating another target for hackers, and another opportunity for introducing fraudulent records, and your second copy will very likely not have the same degree of security as your primary store.

#### Issuance Log

You'll most likely want to keep a record of the credentials you've issued, whether for reporting, legal requirements, to manage revocation, etc.

You could store this data in your existing institutional store - and this might often be the best choice given that your institutional store likely has robust security - but sometimes changing the structures of your institutional store can be difficult both operationally and politically.

A reaonsable compromise can be to store minimal details about issuance in a completely separate database. Ideally you'd store pseudo-anonymous data from which full data can later be recovered as needed. For example, for each issuance you could potentially record some new unique id (like a UUID) along with the date of issuance and some identifier from your institutional store from which you could later pull the data needed to fully identify the record issued. The point here is that if your issuance log were compromised, you'd not directly expose private information.

### Authentication

Here we are talking about authenticating the holder of the credentials, so for example, an alumni of a university or college who earned a degree or a certificate.

There are again two parts to this:

- Authenticating a holder when they *collect* a copy of their credential
- Authenticating a holder when they *present* credentials for verification

##### Authenticating at collection time

When giving out a credential you want to know that the person collecting the credential is the intended recipient. There are a few ways of doing so.

###### Existing Identity Provide (IdP)

If there is an existing IdP (Identity Provider, i.e, a username/passwrod authentication system, like Microsoft Auzure) that you can use to authenticate holders when they collect a copy of their credential, then you're well on your way - you could authenticate your collection page in the same way you authenticate other pages in your system, and you've likely got plenty of local expertise in doing this. This approach can also potentially work very well with your institutional data store because the IdP will likely provide a token that can be used to ensure that just the data for that user is pulled.

If, however, you don't have an existing IdP, or say a student's campus account is closed when they graduate, then you'll need to look for a different approach to authenticating holders when they want to collect a copy of their credential. Some options:

###### Email Verification

One alternative is to have a credential request page at which the holder can ask that an email be sent to a given email address. If your system finds a match for that email address in your student records then the system emails out a link from which the holder can collect the credential. This works much like the password recovery systems that email a password reset link to the email address on record for a given account. In both cases, the links sent out contain a token that when submitted (by clicking the link) effectively confirms that it is the student collecting the credential - because only they would have receieved the email with the link/token.

###### Identify Proofing

If you don't have a current email address on file for the holder, you'll likely then need to verify their identity in some other way.  This could be a web page that asks for a combination of details only they would know - say their student number, the year they graduated, the residence in which they lived, the name of the instructor they had for a given course, etc. If this isn't viable, or doesn't fit with institutional policy, you may need to verify their identity in-person or over the phone.

In either case, after verifying identity, the credential could be directly sent to the recipient, or possibly better, they can add their email address to the system, and then go back to an email verification page to collect their credential. The advantage to collecting their email adress is that the next time they collect another credential or a replacement for the same credential, they can simply use their email address without having to go through the proofing step again.

##### Authenticating when presenting credentials (holder binding)

This is a very interesting topic.  When a holder presents a credential - say a student providing proof of their degree to a job recruiter - how does the recruiter know that the credential belongs to the holder?

The obvious check is against the holder's name or some other similar identifier. If the name on the credential matches the name of the person providing the credential then we assume the credential belongs to that person. We usually know the name of the person from other contextual information, for example, corroborating identification like a driver's licence or credit cards.

Another option is to include an email address - belonging to the holder - in the credential. A verifier could then send an email to that address asking for confirmation.

There is also another way we can enable a holder to prove that the credential belongs to them. We can use what is sometimes called a cryptographic 'holder binding'. Two advantages of the cryptographic holder binding are:

1. It doesn't need to reveal private information about the holder (like their email address or other information that might be found in corroborating evidence like a driver's licence)
2. It can be in some cases be verified automatically and instantly.

The way the holder binding works is that when we issue a credential to holder, we have usually already authenticated the holder somehow (as described in the prior section [Authenticating at collection time}(#authenticating-at-collection-time)). So as part of that authentication session, we can ask the holder to give us a public key (or a [DID](https://www.w3.org/TR/did-core/)) for which the holder has the corresponding private key (which they must, not surprisingly, keep private). We can then add that public key (or DID) to the credential and sign the entire credential.

This effectively then declares to anyone who later wants to verify the credential that we (the issuer) attest that the public key/DID in the credential does belong to the holder (because we got it from them in some authenticated way).

So now, when the holder presents the credential to someone else for verification, the verifier can give the holder (really, their wallet) a challenge (just a random string of some sort, like 'k2kn35l') that the holder (really, their wallet) can sign with their private key and return to the verifier. The verifier's software then confirms that the challenge was signed by the private that corresponds to the public key in the creential. 
The upshot is that the holder thereby confirms that they 'control' the corresponding public key in the credential, and therefore also the credential itself.

TODO:  add a diagram

Adding a holder binding makes things a bit safer, kind of like the second factor auth we've all come to use, and can also help with identity theft - if someone has gotten other credentials of mine they might be able to use those to impersonate me, but if they are asked to demonstrate control of those credentials through a holder binding challenge, then impersonation becomes more difficult.

Another advantage to a holder binding is that they can used to prove control of a credential without having to provide corroborating evidence like a driver's licence. This might be useful when a credential is used in some automated way - say as a credential authorizing us to operate a 3-D printer in a library. In that case the printer can't as easily ask us for a driver's licence or send us an email, but could automatically issue us a challenge (via our wallet) which the wallet can sign on our behalf using our private key. 

It can be tricky to collect the public key from a holder before issuing them a credential, but fortunately, the DCC provides an 'exchange' service that you can incorporate into your issuance. The DCC exchange service works with the Learner Credential Wallet and together the two take care of:

- prompting the LCW to send the holder's DID on their behalf, including asking the LCW to use the holder's private key to sign a challenge
- verifying the DID
- adding the DID to the credential before signing
- signing the credential
- returning the credential to the LCW

An example of how to use this is in our [Exchange Coordinator](https://github.com/digitalcredentials/exchange-coordinator).

### Revocation

You may want to revoke credentials for a number of reasons - mistakes, changes to entitlement, revoked priviliges, name changes, and so on. You may also, of course, not be concerned about revocation, say for certificates of attendance.

With a conventional centralized system, credentials are verified by calling back to a central system or database to confirm that the credential is legitimate, and in this case the central system can tell us at that moment that the credential is valid, or no longer valid (it has been revoked).

But, one of the important characteristics of verifiable credentials is that they should not be verified by a call back to a centralized system. Verifiable Credentials are instead verified by checking the cryptographic signature on the credential, which can be done anywhere, by anyone, with no need to call the central server. They are decentralized, removing dependence on the central system, and notably removing an opportunity for the issuer to control or monitor credential usage.

Since we don't make a call back to the central server to verify the credential, we also don't want to make a call bqck to check for revocation. So how do we then know if it has been revoked? One approach is to publish a list of revoked credentials where the list simply contains anonymous (or pseudo-anonymous) ids of all revoked credentials. The verifier retrieves the list and checks to see if the credential is in the list. Crucially, the issuer can't know which credential is being verified because we retrieve the entire list. 

The list can also be published in different places for redundancy, and also to make it harder for the issuer to monitor access to the list. The list could, for example, be published to a cdn.

The DCC advocates using a subtle variation to a simple list called a [Bitstring Status List](https://www.w3.org/TR/vc-bitstring-status-list/). Instead of simply adding ids to a list, we create a list in advance, where the list is a [bitstring](https://en.wikipedia.org/wiki/Bit_array) - effectively a string of zeros and ones, like so:

```00000000100000010100```

Each character position in that string is assigned to a single credential when the credential is issued. If the character at that position is a zero then the credential is active (hasn't been revoked). If the charcter is a one then the credential has been revoked.

So a verifier checking a credential has to download the right bitsting and then check the position in the string that has been allocated to the credential being checked.

A major advantage to the bistring status list is size - with only a single bit for each credential and with the advantages of compression, a very large list can compress down to a very small size.

The DCC provides two implemenations of a bistring - one that stores the bitstring in github and one that stores the bistring in a mongo db. Both implementations are packaged as a microservice that can be wired into a credential flow. We talk more about this service later.

### Public Key Authenticity - Registries and Certificates

A fundamental problem with all cryptographically signed credentials is knowing who signed the credential. 

Anyone can easily generate a new key pair and sign a credential, so how do we know who generated (and used) the key pair that signed a given credential? I could, for example, create a Verifiable Credential that says I graduated from Oxford and sign it with a keypair I've just generated. A verifier would confirm that the signature was valid - that nothing in the credential had been tampered with - and that the credential had been signed by the private key associated with the indicated public key. How then does the verifier know that it wasn't Oxford who signed it?

The answer is that a verifier must have some trustworthy way of knowing which public keys belong to which organization. Two such ways are:

- a registry of signing keys (or DIDs)
- certificates given to issuers by an overarching authority

#### Registry

Oxford would add the public key part of their keypair to a well known registry - which is fundamentally just a list of public keys. The verifier would check the registry to confirm that the public key associated with a given credential was in the registry.

The registry of course has to be very carefully managed so that fake keys can't be added. Any new submissions or changes must be carefully vetted. These lists are public, though, and so subject to public scrutiny. It is not hard to imagine then that each institution would regularly confirm that only it's own legitimage keys are registered in its name. Nevertheless, we do have to be careful about adding new keys.

Note that a signed credential can't itself point at the registry because then a fake credential could simply point at a fake registry. The verifier has to know which registires are legitimate - independently of anything in the credential.

Registries are still in their early days. It has yet to be worked out who will maintain registries, although a likely guess is that entities that already manage membership or accredation will manage the keys for its membership. 

For now the DCC has set of registries to which you can add your DIDs:

* [sandbox registry](https://github.com/digitalcredentials/sandbox-registry)          
* [dcc pilot registry](https://github.com/digitalcredentials/dcc-pilot-registry)
* [community registry](https://github.com/digitalcredentials/community-registry)         `
* [issuer registry](https://github.com/digitalcredentials/issuer-registry)

These registries are checked by the [Learner Credential Wallet](lcw.app) and by [VerifierPlus](https://verifierplus.org) to verify the authenticity of the signing key. The wallet and verifier fundamentally just look to see if the DID used to sign a given credential is listed in one of the registries. If it is, they'll say which one. If not, they'll the key isn't known.

Initially you'll want to get your key into the sandbox registry, which means adding it to this file:

```[https://github.com/digitalcredentials/sandbox-registry/blob/main/registry.json](https://github.com/digitalcredentials/sandbox-registry/blob/main/registry.json)```

Follow the pattern in the file to add your DID, and then submit a pull request. If you need help with that let us know.

Note too that if you have your own verifier - say a page on your web site at which your own issued verifiable credentials can be verifier (like say certificates your school has issued) then you will know which are your own public keys - you effectively are maintaining your own registry. You still have to be careful to build that check into your verification though - be sure to check that the public key associated with a credential is in fact one of your own.

We've talked about public keys here, but more likely is that you'll use a [Decentralized Identifier (DID)](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjTnLn24LeEAxU1C3kGHSTVAngQFnoECBMQAQ&url=https%3A%2F%2Fwww.w3.org%2FTR%2Fdid-core%2F&usg=AOvVaw2lmH5NVEyOOCzXxjwUG8Yh&opi=89978449) to manage your issuing keys, and at the moment, likely a did:web DID, which helps manage key rotation. The upshot is simply that the verifier confirms that a given DID is in the registry, rather than the public key. The verifier still, though, has to confirm that the given DID also manages the actual public key involved.

#### Certificates

An alternative to a registry is for an overarching central authority to provide certificates (which can themselves be Verifiable Credentials) to issuers where the certificates assert that a given issuer is authorized to issue credentials using a given key or DID. When the issuer then issues a credential, it includes the authorizing certificate in the issued credential, or points to where it can be found. In this case the verifier will now have to confirm that the authorizing certificate is valid (and hasn't itself been reovked). 

For the moment, the DCC is using registries rather than authorizing certificates.

### Private Key Security

Your private key - the 'half' of a keypair, that does the signing - must be kept as secret as possible. Ideally you in fact don't ever want a human to see it. The risk is that if someone copies your signing key they can then issue credentials that will verify as authentic. And unless you happen across credentials that were fraudulently issued, you may well never know that your key has been compromised or that fraudulent credentials were issued with it.

There are a few options for securing your signing keys from least to most secure. Which you choose will likely depend on the type of credential you are issuing and institutional policy. For testing and evaluation, using an environment variable is the easiest choice, and is how the DCC signing-service is configured by default.

#### Environment variable

Have someone manually generate the key and set it in an environment variable. With the DCC software, the key is specified in the environment variable as a deterministic seed (deterministic meaning that one seed corresponds to only one keypair).

#### In-memory

Have your system code automatically generate a new key pair when the system starts or restarts, and possibly also at some regular interval like once a month.

The private part of the key pair will exist only in system memory and will never be written out anyhow. When the system shuts down, or the key is deliberately replaced, the old private key permanently disappears.

Your code will also have to publish (to a [registry](#public-key-authenticity-registries)) the new public key every time a new key pair is generated.

#### Hardware Security Module

At the moment, the most secure option is the use a [Hardware Security Module (HSM)](https://en.wikipedia.org/wiki/Hardware_security_module). The HSM generates and manages keys, and signs things for you when needed.

An HSM does incur a cost, either for the hardware if you buy one to use in-house, or often per transaction if someone else manages it for you.

Whether or not to get an HSM often will depend on how concerned you are about key theft.

#### Mitigating Stolen Signing Keys

There are a few things you can do to mitigte the risk of a stolen key:

##### Rotate keys frequently

By frequently switching your key you restrict the period in which a stolen key can be used. After the key has been "rotated out" (retired, decommissioned) the key can no longer sign new credentials.

There are essentially two ways to retire a signing key:

* Register the period during which the key was active. This will only work, though, if you have a trustworthy timestamp attached to your credentials so that you can confirm the credential was signed during that period.

* Revoke the key entirely. Any credentials signed with the key are no longer valid. You could look at this as another way to set an expiry date on the credentials themselves. This can be a reasonable approach if you provide a convenient way for holders to refresh their credentials or retrieve new copies.
  
You can read more about how DIDs manage key rotation and key revocation here:

* [https://www.w3.org/TR/did-core/#verification-method-rotation](https://www.w3.org/TR/did-core/#verification-method-rotation)
* [https://www.w3.org/TR/did-core/#verification-method-revocation](https://www.w3.org/TR/did-core/#verification-method-revocation)
 
##### Expire credentials aggressively

You can set the [https://www.w3.org/TR/vc-data-model/#expiration-0](expirationDate) property in the verifiable credential itself (NOTE: expirationDate will be replace by [validUntil](https://www.w3.org/TR/vc-data-model-2.0/#validity-period) in version 2 of the Verifiable Credential specification. 

This also only works if you have a trustworthy timestamp attached to the credential. You can, however, also expire credentials, without using a timestamp, by expiring the signing key as described in the prior section on key rotation.

##### Public hash registry

Mention how a public hash registry can help detect fraud????

### Wallets

Wallets are applications in which holders store their credentials. For the moment there are two kinds of wallets:

#### App Wallets

These are apps on a device like a mobile phone. The [Learner Credential Wallet](https://lcw.app) is one example.

A significant advantage to app wallets are that the holder more directly controls their credentials because the credentials are stored directly on the phone.

A disadvantage is that if the phone is lost, it may be inconvenient, difficult, or even impossible to replace the lost credentials. Wallets like the [Learner Credential Wallet](https://lcw.app) do provide backup, but at the moment backup is a manual process and may be neglected. Usually - but not always - fresh copies of the credentials can be retrieved from the issuers.

#### Web Wallets

Web wallets are wallets that typically store your credentials ... more to come.

### Verification

Verification of credentials can happen in any number of places:

#### Wallets

Wallets like the [Learner Credential Wallet](https://lcw.app) verify credentials when first adding them and also when displaying them. This verification assures the credential holder that their credentials are valid (and continue to be valid,e.g, they haven't expired).

Wallet verification should not be used to show someone else that your c

#### Web Pages

Web apps like [VerifierPlus](https://verifierplus.org) will verify a verifiable credential that has been uploaded or pasted into a form. Veification pages like this would typically be used by someone to whom a credential has been given as proof of something. A job recruiter, for example, to whom a job candidate has emailed a copy of their degree.

A web verifier like [VerifierPlus](https://verifierplus.org) can also scan a QR (using the phone or laptop camera) for a credential that has been presented from a wallet like the  [Learner Credential Wallet](https://lcw.app).

#### Verification Apps

At some point there might be dedicated apps on phones that can scan a QR (possibly presented to them from the wallet on another phone) and verify authenticity. For the moment, web verifieers like [VerifierPlus](https://verifierplus.org) can do that.

#### Automated Verification

Once verifiable credentials achieve some amount of critical mass, organizations will automatically verify credentials using internal verification code. A site like [ORCID](https://orcid.org) for example would automatically verify any credentials uploaded to an ORCID profile. Similarly, a job recruitment site could verify any uploaded credentials (like a degree).

### Credential Display

Credential display is for the moment almost always part of either verification or display in a wallet.  The verification application or wallet decides what to show, which is usually some fairly generic set of fields like credential name, issuance date, expiration date, issuer logo, and credential logo. Active work is underway (Spring 2024) to enable customized display using...

#### renderMethod

Under active development is a method that allows credentials issuers to provide a template for rendering their credentials a certain way. The [Learner Credential Wallet](https://lcw.app) has some support (in development) for generating PDFs from HTML templates specified in a renderMethod. 

### DCC components and services

Okay, now we can talk about the components and services of the Digital Credentials Consortium.

#### Services Overview

The DCC has split out issuance into a number of components that can be assembled and configured to suit the needs of individual projects. The core of the components are microservices, each microservice running as a nodejs express app, and different services 'wired' together by a 'coordinator', which is another nodejs express app itself. The coordinator and the services it coordinates all run together in a docker compose network, defined by a docker compose file.

The docker compose file is actually the key piece that we anticipate most projects will use, and in many cases a project wanting to run our services could have nothing more than a single docker compose file with all the specific project configuration within the compose file and environment variables.

The compose file simply defines the set of services to be used, where each service definition simply points at the relevant image in docker hub.

So you needn't checkout github repositories - just create a docker compose file with your configuration set as environment variables.

##### Services Versioning

We use [semantic versioning (semver)](https://semver.org) for our services. We publish new images to docker hub with semver tags that correspond to github tags in the corresponding repository.

#### Issuer coordinator

[Github repo](https://github.com/digitalcredentials/issuer-coordinator)

This is our simplest coordinator. It does two things:

* signs credentials
* allocates a status position for later revocation

The status position allocation is optional.

##### When you'd use it

You'd use this coordinator when you handle everything else in your local environment besides signing and status allocation, including:

* managing authentication
* data storage
* construction of the verifiable credential to be signed
* exchange with holder or with wallet

Basically, you retrieve the data from your stores, build a VC, and then simply pass the VC into the coordinator,
which will sign it, optionally allocate a status position, and return it. You then take care of getting the VC to the holder.

##### How you'd use it

Simply start up the coordinator in docker compose - as described above in [Simple Signing Demo](#simple-signing-demo) and more thoroughly in the [Github repo](https://github.com/digitalcredentials/issuer-coordinator) for the issuer coordinator. You'll have to set a couple of environment variables for your signing keys and your github or mongo store (if you are using either), but that is pretty much it. Then you can simply post an unsigned verifiable credential to the service endpoint when you need to sign.

#### Exchange coordinator

[Github repo](https://github.com/digitalcredentials/workflow-coordinator)

This is a more complex scenario. The exchange coordinator wires together three services:

* [signing-service](#signing-service)
* [status-service-github](#status-service-github) OR [status-service-github](#status-service-mongo)
* [transaction-service](#transaction-service)

NOTE: the status-services are optional and aren't required if you don't intend to revoke credentials.

The exchange coordinator differs from the [issuer-coordinator](#issuer-coordinator) is that it uses the transaction-service to manage an exchange with a wallet. The exchange is based on the [VC-API spec](https://w3c-ccg.github.io/vc-api/#exchange-examples) and specifically uses a [DIDAuthentication Verifiable Presentation Request](https://w3c-ccg.github.io/vp-request-spec/#did-authentication) in which the issuer asks the wallet for a DID belonging to a holder, along with proof that the holder owns that DID. The wallet supplies the proof by signing - using the private key associated with the holder DID - a 'challenge' that the issuer gives the wallet.

Once the issuer has got the holder's DID the issuer adds that DID to the VC, signs it and returns it. The VC is now 'bound' to the holder in the sense that the holder can subsequently prove they own the credential by signing challenges with the private key for the holder DID that was included in the VC. 

##### When you'd use it

When you want holders to add your credentials to a wallet like [Learner Credential Wallet](https://lcw.app), and more specifically so that you can add a 'holder binding' to the credential.

##### How you'd use it

The exchange coordinator is meant to be integrated into an existing environment. One approach is to spin it up in a docker compose like this [one (TODO - add docker compose)](#exchange-compose-demo), and then from your own system, post populated but unsigned VCs to the /exchange endpoint which will return a deeplink that you can then give to the credential recipient, either by emailing it to them, or presenting it directly to them on a screen either as a link (if they are on their phone) or as a QR (if they are on a laptop) that they can scan from their phone. The rest of the exchange with the wallet is taken care of by the exchange coordinator.

So in summary, to use the exchange coordinator, you:

* Pass it an unsigned (but otherwise complete) Verifiable Credential
* The coordinator replies with a deeplink that you can give to the student to collect the credential

#### signing-service

[Github repo]([https://git](https://github.com/digitalcredentials/signing-service)

Fundamentally simply takes an unsigned [Verifiable Credential](https://www.w3.org/TR/vc-data-model/), signs it, and returns it.

##### Why you'd use it

To sign Verifiable Credentials.

##### How you'd use it

You can use this alone, but if you intend to use it with one of our status services and/or as part of a wallet exchange then you likely want to look at one of our 'coordinators':

* [Exchange coordinator](#workflow-coordinator)
* [Issuer coordinator](#issuer-coordinator)
  
#### status-service-github

[Github repo]([https://git](https://github.com/digitalcredentials/signing-service-github)

This is an implementation of the [Bitstring Status List](https://www.w3.org/TR/vc-bitstring-status-list/) specification, using Github for storage.

##### Why you'd use it

To allocate a status position that can later be 'flipped' to revoke the credential.

You might consider using this implementation of the status list, rather than our mongo implementation, if you have a relatively low number of credentials. If you are getting into the thousands of credentials then you could run into rate limiting problems with Github.

This implementation might also be a good choice when first starting out, as relatively simple means to get a status system running, particularly for experimentation or evaluation.

This could also be a good choice for a one-off credentialing project, like say where you wanted to issue certificates to a few hundred conference attendees, and didn't want to have to maintain anything beyoned that issuance. Because the status list is in github, you could just let it live there forever more. In this case you may, of course, not really care about being able to revoke the credentials, but if you did, the github implemenation of the status list could be a good choice.

###### Advantages

* relatively easy to setup
* no local infrastructure needed
* the durability of Github (rarely goes down)

###### Disadvantages

* You'll potentinally run into rate limiting with Github if you have a significant number of credentials (thousands)
* You are dependent on Github
  
##### How you'd use it

You can use this by itself to allocate a status position, but you probably want to look at how we use it in a coordinator, as described in the sections above. The coordinators effectively take care of calling the status service for you.

#### status-service-mongo

[Github repo]([https://git](https://github.com/digitalcredentials/signing-service-github)

This is an implementation of the [Bitstring Status List](https://www.w3.org/TR/vc-bitstring-status-list/) specification, using Mongo for storage.

##### Why you'd use it

To allocate a status position that can later be 'flipped' to revoke the credential.

You might prefer this implementation to the github version if you've got a lot of credentials that would bump up against Github rate limits, or you just don't want to depend on Github.

This implementation might also make sense if you already use Mongo for other things and adding a db incurs little extra cost or maintenance.

Note too that an option is to run a local Mongo instance directly in your docker compose.

###### Advantages

* scalable (and in particular much moreso than our github implementation)
* can be used with a locally running mongo or a cloud based mongo
* no dependency on external company (if you run a local mongo instance)

###### Disadvantages

* You are on the hook for keeping your implementation running for the lifetime of the credentials.
* Extra complexity of running a Mongo instance. A cloud hosted version could help, with could incur hosting charges.

##### How you'd use it

You can use this by itself to allocate a status position, but you probably want to look at how we use it in a coordinator, as described in the sections above. The coordinators effectively take care of calling the status service for you.

#### transaction-service

[Github repo]([https://git](https://github.com/digitalcredentials/transaction-service)

Manages the transactions involved in the VC-API exchange between a wallet and the issuer, specifically:

- stores data associated with a [VC-API exchange](https://w3c-ccg.github.io/vc-api/#initiate-exchange) and generates an exchangeID and transactionID
- generates and verifies UUID challenges used in [DIDAuthentication Verifiable Presentation Requests](https://w3c-ccg.github.io/vp-request-spec/#did-authentication)
- verifies [DID Authentication](https://w3c-ccg.github.io/vp-request-spec/#did-authentication) signatures
- generates both DCC deeplink and [CHAPI}(https://chapi.io) wallet queries

##### When you'd use it

You'll use this if you want to directly exchange credentials with a wallet like the [Learner Credential Wallet](lcw.app). You will probably always use this with the [Exchange Coordinator](#exchange-coordinator) - or a variant - and so therefore to manage the exchange with a wallet that supports either [VC-API exchange](https://w3c-ccg.github.io/vc-api/#initiate-exchange) and/or [CHAPI}(https://chapi.io) wallet selection.

##### How you'd use it

Typically as part of a coordinator like the [Exchange Coordinator](#exchange-coordinator), which you can take a look at for an example of usage.

#### admin-dashboard

[Github repo]([https://git](https://github.com/digitalcredentials/admin-dashboard)

This is a set of web pages backed by a mongo data store for managing a collection of credentials. It allows:

* uploading a CSV containing data for a 'batch' of credentials
* setting up templates for populating a  [Verifiable Credential](https://www.w3.org/TR/vc-data-model)
* setting up templates for populating emails sent to recipients
* sending emails to recipients in a given batch
* recipients to collect credentials from a collection page that adds credentials to the [Learner Credential Wallet](https://lcw.app)

It is based on the [Payload framework](https://payloadcms.com).

##### When you'd use it

If you intend to issue your credentials in batches, want automated email notifications sent to recipients, and want to use the [Learner Credential Wallet](https://lcw.app) for credential collection.

If, however, you'd like to integrate more directly into your existing institutional systems, you'd want to look at using the services more directly. You might want to take a look at our [simple signing demo](#simple-signing-demo) to get a sense of how this could work.

###### Advantages

* you can get up and issuing credentials fairly quickly
* can be used with a locally running mongo or a cloud based mongo
* UI tools for setting up, reviewing, and running batches and for viewing issuance records
* Built-in batch mailing to notyify recipients that credentials can be collected

###### Disadvantages

* If you store credential data in an existing institutional store then you'll be creating a second copy in the admin-dashboard's mongo store.
* Only works with Mongo.
* Requires manual CSV uploads for every batch
* Opinionated configuration as far as batch structure, templating, csv upload, email sending, and use of the [Learner Credential Wallet](https://lcw.app)

##### How you'd use it

A good example of usage is in the  [admin dashboard demo](#admin-dashboard-demo).

#### Collection Page

[Github repo]([https://git](https://github.com/digitalcredentials/admin-dashboard)

NOTE: the collection page is part of the same github repo as the admin-dashboard, but is in fact a separate app with its own Docker file, and is meant to run as a separate service, usually within a docker compose.

This is a simple web page (written with [Astro](https://astro.build)) from which credentials are collected. It is intended to be used with the admin-dashboard. If you aren't using the admin-dashboard, the collection page won't be directly usable, but it might help as a guide when designing your own page. You can certainly subsitute your own collection page as a separate web app running within the docker compose (or even outside it).

##### When you'd use it

This page pretty closely depends on the admin-dashboard, so you'd use this as part of the admin-dashboard, although you could substitute your own page.

##### How you'd use it

Typical usage would be to include the webapp in a docker compose, like we do in the demo [admin dashboard demo](#admin-dashboard-demo) compose file.

#### verifier-service

[Github repo]([https://git](https://github.com/digitalcredentials/verifier-service)

In the works. It will allow posting credentials for verification, following the [VC-API specification](https://w3c-ccg.github.io/vc-api/)

##### When you'd use it

If you need to verify credentials. Maybe, for example, as an extra check during credential issuance to proactively confirm that the issued credential properly verifies.

##### How you'd use it

Run it as a web service, either directly or from docker or docker compose, and POST credentials to it for verification.

### Learner credential wallet (LCW)

[Learner Credential Wallet](lcw.app)
[Github repo](https://github.com/digitalcredentials/learner-credential-wallet)

## How to get started

If you are fairly new to digital credentialing, you might want to start with the two hands-on demos we describe up in the [Overview](#overview) section. Both demos will help you understand how we typically manage a credentialing deployment using a docker compose file that 'wires' together a few services using a 'coordinaotr'. 

The [Simple Signing Demo](#simple-signing-demo) will give you a good sense of what a Verifiable Credential is, what the signature looks like, and how you can use our signing service.

The [Admin Dashboard Demo](#admin-dashboard-demo) will give you a sense of:

* what the data that goes into a credential typically looks like
* how a credential recipient can collect a credential from an email and wallet
* one way you might batch your issuances (with a CSV upload)
* how you can manage revocation of credentials

Trying both demos should altogether take likely less than 30 minutes.

From here your choices will essentially depend on how you'd like to tie into your existing system.

### Scenarios

#### Do you want to issue just a very few 'one-time' credentials?

You could essentially just use the simple issuer demo compose we've provided, constructing your own verifiable credentials (which are fundamentally just text files) and simply post them to the signing service to get back the signed credential. You could run it fairly easily on your laptop.

You'd additionally have to:

- generate and set your own keypair as described in the signing-service (TODO: add link)
- register the public key part of your keypair with a registry like the DCC registry (TODO: add link)

Altogether expect to spend anywhere from a half hour to a few hours depending on how deeply you'd like to dig into things.

#### Do you want to be able to revoke your issued credentials?

If you want to be able to revoke the credentials, you'll need to enable the status service, as described in the [signing-service](TODO: add link here). You'll also have to choose between the github and mongo implementations of the status service.

The github implementation is fairly easy to setup and can mean less ongoing maintenance. The major downside is that github has rate limits, so if you've got a lot of credentials this implementation might not be feasible. Another downside is that you may not be comfortable depending on Github in the long term.

The mongo implementation requires a mongo setup. You can use a cloud based mongo service (like MongoCloud), a local Mongo setup, or even a mongo image running as a service within your docker compose. [TODO add this compose option]

#### Do you want to control how your credential appears in a wallet or when verified?

Verifiers and wallets for the moment are configured to show just a few basic fields like the name of the credential, name of the issuer, name of the recipient, date of issue, date of expiry, issuer logo, small credential image, and maybe a description and criteria. They are typically displayed the same way for all credentials.

There will likely come a time when wallets recognize certain types of credentials and can display them appropriately, like an undergraduate degree or a course credential.

You may, though, want to control the display of your credential now, or may have specific requirements - you may, for example, want your university degree to look identical to the paper diploma.

A relatively new spec called the [vc-render-method](https://w3c-ccg.github.io/vc-render-method/) provides a way to define how you'd like the data in your VC presented, or 'rendered'. This could be visually, audibly, or haptically (as touch). 

The DCC for example has just released (March 2024) a version of the DCC [Learner Credential Wallet](lcw.app) that provides an option to generate a PDF for a credential if the credential includes a render method that specifies an HTML template where the template includes 'slots' into which the data of the credential (e.g. recipient's name) can be inserted at display time. Using the generated HTML, the LCW then generates a PDF for export and sharing.

Rendering is relatively new, so you'd likely want to talk to us about your needs.

#### Do you want a standard data structure for your credential?

You may be issuing credentials with a fairly standard data model, like a university degree or a college diploma. And you likely want to ensure your credential is compatible, or interoperable, with credentials issued by other institutions. 

The DCC is working with the community towards defining standard data models for certain credentials, in particular the degree or diploma. We expect to soon have another microservice to provide data 'templating' for certain kinds of credentials. In this case you would simply pass in the 'raw' data for a credential (issuer name, holder name, date of issue, etc.) and get back in return a populated verifiable credential with a standard data model for the given credential 'type'.

Until templating is available, please get in touch if you've got a credential that you feel is common enough to benefit from a shared template.

#### Do you want a complete independent credentialing system to which you upload data for credentials?

This is our [Admin Dashboard](https://github.com/digitalcredentials/admin-dashboard) which provides CSV upload of data, generation of VCs using templates, email notification to recipients, wallet integrated collection, and revocation.

Some of the reasons you'd use this system:

- to keep your digital credentialing separate from other systems
- you have a fairly simple batch of credentials you'd like to issue and don't want a lot of setup
- experimentation
- just want to get up and running quickly

#### Would you like to tie directly into your institutional store so that when recipients collect credentials, the credentials are dynamically generated using data from this store?

In this scenario, you want a more integrated solution to streamline processes and reduce duplication. You've got a few choices in this case, but fundamentally, you'll run either the issuer-coordinator or the exchange-coordinator alongside your other systems, and then at collection time (when a recipient comes to collect a credential) you'll simply POST (as an http call) the data to one of the two coordinators. 

The issuer-coordinator will simply return a signed credential, with a revocation status allocated if requested, and you'll pass that on to the recipient. You'll handle all authentication around the request (for example using your institutional IdP) as well as the UI (i.e, the page from which the student collects the credential).

The exchange-coordinator will return a link that you can provide to the student, from which the student can collect their credential. It is up to you to provide the UI for the collection page. Again, you'll handle authentication of that page.

#### Do you want to issue credentials to the DCC Learner Credential Wallet?

In this case you can again use either the issuer-coordinator or the exchange-coordinator but the interaction with the wallet varies from one to the other.  

With the issuer-coordinator, the recipient will have to themselves 'import' the credential into the wallet either by copy/pasting the JSON text of the credential, or by scannning a QR encoding of that same JSON text (where you would create the QR), or by downloading the credential from a link if you opt to provide a hosted link for the credential.

The exchange-coordinator on the other hand manages the exchange with the wallet. When you post the data for a credential to the coordinator, it returns a link that you give to the student. When the student clicks the link (or scans it as a QR) it opens the Learner Credential Wallet and takes care of the rest of the exchange between the wallet and the coordinator. The end result of clicking the link is that the student's credential is added to their wallet. The exchange-coordinator therefore provides a simpler and more intuitive experience for the student.

#### Do you want to add a holder binding to your credentials. (A holder binding allows the holder to later prove they 'control' the credentials, in some sense like having a password).

If so, you'll want the exchange-coordinator which manages this binding. Note that the admin-dashboard uses the exchange-coordinator and so provides this binding.

#### Do you want to notify recipients by email that they can collect a credential?

If you opt to use the DCC admin dashboard then emailing is managed for you. You will need an email service like sendGrid or even your existing institutional email system, but you need only give the dashboard the login and password for that.

If you don't use the dashboard then you'll have to take care of constructing, populating, and sending the emails. 

There are a few ways you can email the student including:

* Send them the fully constructed verifiable credential. They'd then have to 'import' it into their wallet if they did want to keep it in a wallet, but they could also 

* Send them a link from which they can dowload the credential.

#### Do you want to use your existing IdP when credentials are collected?

#### Do you want to allow collection via a token sent in an email?

### Docker Compose Examples

We've put together examples of docker compose files for various configurations. These are all meant to be examples that you'll likely customize, but they should be pretty close to what you'll deploy in the end.

#### Simple Issuer

This is the same compose file described in the [Simple Signing Demo](#simple-signing-demo) at the start of this guide. It simply runs an issuer that takes an unsigned verifiable credential, signs it, and returns it. See the [Simple Signing Demo](#simple-signing-demo) for an example of how you'd use it.

[simple-issuer-compose.yml](docker-compose-files/simple-issuer-compose.yaml)

With docker installed and running, start up the issuer with this one-liner:

```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/simple-issuer-compose.yaml | docker compose -f - up```

#### Local dashboard

This will run the [Admin Dashboard](https://github.com/digitalcredentials/admin-dashboard) and all supporting services locally on localhost (e.g., on your laptop).

[dashboard-dns-compose.yaml](docker-compose-files/dashboard-dns-compose.yaml)


```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/admin-dashboard-compose.yaml | docker compose -f - up```

Once it has started up, you can experiment with the locally running system at [http://localhost:3000](http://localhost:3000) and in particular take a look at our [Admin Dashboard Getting Started Guide](https://github.com/digitalcredentials/admin-dashboard/blob/main/docs/GETTING_STARTED.md)

#### Dashboard with DNS

This will run the [Admin Dashboard](https://github.com/digitalcredentials/admin-dashboard) and all supporting services on an instance that's got a domain name. You simply supply the domain name as an environment variable when running docker compose, and the compose file will take care of configuring and running nginx including generating and maintaining a certificate.

[dashboard-dns-compose.yaml](docker-compose-files/dashboard-dns-compose.yaml)

You can run it like so, subsituting your domain name for the value of the HOST environment variable:

```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/dashboard-dns-compose.yaml | HOST=myhost.org docker compose -f - up```

