# Digital Credentials Consortium Deployment Guide

Setting up digital credential issuing typically requires integrating into existing environments with varying requirements. This guide will help you choose and configure Digital Credentials Consortium issuing components for your requirements.

## Table of Contents

- [Overview](#overview)
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
  - [Overview](#overview)
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

- [License](#license)

## Overview

The [Digital Credentials Consortium](https://dcconsortium.org) provides open source software for issuing [Verifiable Credentials](https://www.w3.org/TR/vc-data-model/). Recognizing that every issuer's environement can be different, we've structured the software as a set of composable services where the services are combined and invoked by a 'coordinator' service. We provide a couple of coordinators that should cover most cases, but you can also create your own to 'wire' together services according to need.  All the services and the coordinators are node express apps exposing http APIs, and are intended to run in a Docker compose network on your servers or servers you control. (NOTE: the DCC does not provide hosting - we strongly recommend that you run these services locally so that you control both your own data and the signing itself.)

We've simplified deployments by publishing the services and coordinators as docker images on Docker Hub, which can be easily run with docker compose, with minimal setup and configuration.

You can in fact try different configurations of DCC services in about five minutes by installing [Docker](https://docs.docker.com/engine/install/) (if you haven't arleady) and then running - from a terminal - one of the following docker compose configs we've defined:

###### Simple Signing

Accepts an unsigned verifiable credential, signs and returns it. Try signing a credential with two commands:

1. Start up the docker compose by pasting this into a terminal prompt and hitting return:

```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/simple-issuer-compose.yaml | docker compose -f - up```

The issuer should start up and you should see a message like:

```
dcc-coordinator-1  | {"level":"info","message":"Server started and running on port 4005 with http","timestamp":"2024-02-17T00:26:48.567Z"}
dcc-signer-1       | Server running on port 4006
dcc-signer-1       | POST /instance/test/credentials/sign 200 60.371 ms - 1699
```

2. Now open up another terminal window and issue a credential by pasting this at the prompt and hitting return:

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

Well done you! You've cryptographically signed a credential.

That particular configuration is wired together by our [issuer coordinator](https://github.com/digitalcredentials/issuer-coordinator). The 'wiring' in this case pretty much consists of simply posting the unsigned VC to the signing-service and returning the resulting signed VC. You can see that wiring [here](https://github.com/digitalcredentials/issuer-coordinator/blob/da3fab1b491d847f5dd42d43e0de1a4e8a22bba5/src/app.js#L77-L78)

###### Admin dashboard

Runs two web apps in addition to the wallet exchanger to allow:

* uploading credentials through a CSV
* creating verifiable credential templates
* creating email templates
* emailing recipients a link from which to collect a credential
* a credential collection page

```curl https://raw.githubusercontent.com/digitalcredentials/docs/jc-compose-files/deployment-guide/docker-compose-files/admin-dashboard-compose.yaml | docker compose -f - up```

Once it has started up, you can experiment with the locally running system at [http://localhost:3000](http://localhost:3000)

The rest of this document explains factors you'll have to consider when choosing and configuring issuing components, followed by a description of the components themselves, some configurations that cover common scenarios, and a guide to setting up your own configuration.

## Definitions

Let's quickly define three fundamental terms:

### Issuer

Whoever issues the credential. A university issuing a degree for example.

### Holder

Whoever 'holds' the credential. A student with a degree for example.

### Verifier

Whoever needs to verify a credential that was shared them. A job recruiter verifying a job candidate's degree for example.

## Deployment Considerations

Okay, directly into it then - there are a few primary factors that influence deployments:

Data Storage
Authentication
Revocation
Public Key Authenticity
Private Key Security
Wallets

And a couple of factors that may be outside your control, but should be considered:

Credential Verification
Credential Display

Let's talk about each in turn...

### Data Storage

There are two parts to this:

***Institutional Data*** - Where your institution stores your **fundamental credential data** (e.g. your student records system)

***Issuance Log*** -  Where you'll store **details about issuance** (e.g., when a credential was collected)

In some cases the two can be combined, but you might prefer not to add issuance details to your institutional system (e.g., campus PeopleSoft system) and instead - for simplicity's sake, or maybe because of institutional policy - keep those details in a separate store, like a standalone Mongo database.

Let's go over the two types of data...

#### Institutional Data 

This is the fundamental authoritative data for your credentials - the [system of record (SOR)](https://en.wikipedia.org/wiki/System_of_record) for your credentials. This will often have been around for decades or even centuries, like say for the list of degrees that a university has issued.

Where your institutional data is stored, and maybe more importantly, how you can retrieve that data, will influence the structure of your issuing system. 

Let's talk about two ways you might access that data...

##### Direct pull

If you can pull student records from the existing system 'on demand' - say when a student logs in to a credential collection page and requests something like a transcript - then you could instantaneously and automatically construct and return credentials on-the-fly.

A direct pull system ***might*** be more complex to setup (although often not), but the medium and long term benefits can be significant. Essentially, after having setup the system, credential holders (students/alumni) can simply download a copy of their credential whenever they like. Which also means that replacement of lost credentials is no longer a complicated issue requiring human mediated requests and lookups - the student/alumni can now simply login to get an immediate replacment, directly and automatically issued from the instutiotnal data.  Same goes for name changes - if someone changes their name (for example if they get married) the name change can be handled by the normal institutional process for a name change, which will end up changing the underlying institutional record. And once that is done the student/alumni then just logs back in and get a fresh copy, pulled from the update underlying data.

##### Standalone system

If you can't easily pull from your institutional store for some reason, maybe because of campus restrictions then you may want to look at a stand-alone system to which you add credentials that you expect will be collected. 

You might add credentials to this standalone system using CSV uploads or maybe even some direct export from your institutional system.

A standalone system can be an easy way to get started with digital credentials, especially if you are initially issuing test credentials, or credentials that don't exist elsewhere, like say a certificate of attendance for a conference.

Three significant downsides to a standalone system are:

1. If the credential data is already stored in some other place (e.g, campus PeopleSoft) then storing a second copy in your standalone system means that you now have two copies of the same data, which can easily get out of sync: the case, say, where someone changes their name in the main campus system, but there isn't an automatic change in your standalone system.

2. Adding new data to the standalone system is likely to be a manual process, which can become very complicated. If you are pulling the data out of an institutional store and then uploading it as CSV to the standalone system, you'll typically have to have various checks to confirm the data was properly pulled and added. Problems with character encoding, changed data structures, and so on can be a real hassle.

3. The data in your institutional store most likely has fairly rigorous security and checks to make sure that there are no:
a) data breaches where private data is stolen
b) fake data isn't added to the system.
By creating a second copy you are creating another target for hackers, and another opportunity for introducing fraudulent records, and your second copy will very likely not have the same degree of security as your primary store.

#### Issuance Log

You'll most likely want to keep a record of the credentials you've issued, whether for reporting, legal requirements, to manage revocation, etc.

You could store this data in your existing institutional store - and this might often be the best choice given that your institutional store likely has robust security - but sometimes changing the structures of your institutional store can be difficult both operationally and politically.

A reaonsable compromise can be to store minimal details about issuance in a separate database. Ideally you'd store pseudo-anonymous data from which full data can later be recovered as needed. For example, for each issuance you could potentially record just some new unique id (like a UUID) the date of issuance and some identifier from your institutional store from which you could later pull the data needed to fully identify the record issued. The point here is that if your issuance log were compromised, you'd not directly expose private information.

### Authentication

Here we talk about authenticating the holder of the credentials, so for example, an alumni of a university or college who earned a degree or a certificate.

There are again two parts to this:

- Authenticating a holder when they *collect* a copy of their credential
- Authenticating a holder when they *use* credentials

##### Authenticating at collection time

When giving out a credential you want to know that the person collecting the credential is the intended recipient. There are a few ways of doing that.

###### Existing Identity Provide (IdP)

If there is an existing IdP (Identity Provider, i.e, a username/passwrod authentication system, like Microsoft Auzure) that you can use to authenticate holders when they collect a copy of their credential, then you're well on your way - you could then authenticate your collection page in the same way you authenticate other pages in your system, and you've likely got plenty of local expertise in doing this. This approach can also potentially work very well with your institutional data store because the IdP will likely provide a token that can be used to ensure that just the data for that user is pulled.

If, however, you don't have an existing IdP, or say a student's campus account is closed when they graduate, then you'll need to look for a different approach to authenticating holders when they want to collect a copy of their credential.

###### Email Verification

One alternative is to have a credential request page at which the holder can ask that an email be sent an email to a given email address. If your system finds a match for that email address in your student records then the system emails out a link from which the holder can collect the credential. This works much like the password recovery systems that email a password reset link to the email address on record for a given account. In both cases, the links sent out contain a token that when submitted (by clicking the link) effectively confirms that it is the student requesting the credential because only they would have receieved the email with the link/token.

###### Identify Proofing

If you don't have a current email address on file for the holder, you'll likely then need to verify their identity in some other way.  This could be a web page that asks for a combination of details only they would know - say their student number, the year they graduated, the residence in which they lived, the name of the instructor they had for a given course, etc. If this isn't viable you may need to verify their identity in-person or over the phone.

In either case, after verifying identity, the credential could be directly sent to the recipient, or possibly better, they can add their email address to the system, and then go back to an email verification page to collect their credential. The advantage to collecting their email adress is that the next time they collect another credential or a replacement for the same credential, they can simply use their email address without having to go through the proofing step again.

##### Authenticating when presenting credentials (holder binding)

This is a very interesting topic.  When a holder presents a credential - say a student providing proof of their degree to a job recruiter - how does the recruiter know that the credential belongs to the holder?

The obvious check is against the holder's name. If the name on the credential matches the name of the person providing the credential then we assume the credential belongs to that person. We usually know the name of the person from other contextual information, for example, corroborating identificationn like a driver's licence or credit cards.

There is, though, another way we can enable a holder to prove that the credential belongs to them. We can use what is called a 'holder binding'.

The way this works is that when we issue a credential to holder, we usually have already authenticated the holder somehow (as described in the prior section [Authenticating at collection time}(#authenticating-at-collection-time)) and so as part of that authenticated session, we can ask the holder to give us a public key (or a [DID](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjO_qriw62EAxU5AjQIHcDVBWkQFnoECAcQAQ&url=https%3A%2F%2Fwww.w3.org%2FTR%2Fdid-core%2F&usg=AOvVaw2lmH5NVEyOOCzXxjwUG8Yh&opi=89978449)) for which the holder has the corresponding private key (which they must keep, not surprisingly, private). We can then add that public key (or DID) to the credential and sign the entire credential.

This effectively then declares to anyone who later wants to verify the credential that we (the issuer) attest that the public key/DID in the credential does belong to the holder (because we got it from them in some authenticated way).

So now, when the holder presents the credential to someone else for verification, the verifier can give the holder a challenge (just a random string of some sort, like 'k2kn35l') that the holder can sign with their private key and return to the verifier. The verifier then confirms that the challenge was signed by the private that corresponds to the public key in the creential. 
The upshot is that the holder thereby confirms that they 'control' the corresponding public key in the credential, and therefore also the credential itself.

NEED A DIAGRAM HERE I THINK

Adding a holder binding makes things a bit safer, kind of like the second factor auth we've all come to use, and can also help with identity theft - if someone has gotten other credentials of mine they might be able to use those to impersonate me, but if they are asked to demonstrate control of those credentials through a holder binding challenge, then impersonation becomes more difficult.

Another advantage to a holder binding is that they can used to prove control of a credential without having to provide corroborating evidence like a driver's licence. This might be useful when a credential is used in some automated way - say as a credential authorizing us to operate a 3-D printer in a library. In that case the printer can't reasonably ask us for a driver's licence, but could automatically issue us a challenge (via our wallet) which the wallet can sign on our behalf using our private key. 

It can be tricky to collect the public key from a holder before issuing them a credential, but fortunately, the DCC provides an 'exchange' service that you can incorporate into your issuance. The DCC exchange service works with the Learner Credential Wallet and together the two take care of:

- prompting the LCW to send the holder's DID on their behalf, including asking the LCW to use the holder's private key to sign a challenge
- verifying the DID
- adding the DID to the credential before signing
- signing the credential
- returning the credential to the LCW

An example of how to use this is in our [Exchange Coordinator]()

### Revocation

You may want to revoke credentials for a number of reasons - mistakes, changes to entitlement, revoked priviliges, name changes, and so on. You may also, of course, not be concerned about revocation, say for certificates of attendance.

With a conventional centralized system, credentials are verified by calling back to a central system or database to see if the credential was legitimate, and in this case the central system can simply tell us at that moment that the credential is no longer valid (it has been reovked).

But, one of the important characteristics of verifiable credentials is that they are not verified by a call back to a centralized system. Verifiable Credentials are instead verified by checking the cryptographic signature on the credential, which can be done anywhere, by anyone, with no need to call the central server. They are decentralized, removing dependence on the central system, and notably removing an opportunity for the issuer to control or monitor credential usage.

Since we don't make a call back to the central server to verify the credential, how do we then know if it has been revoked? One approach is to publish a list of revoked credentials. The list simply contains the ids of all revoked credentials. So, the verifier just retrieves the list and checks to see if the credential is in the list. The issuer can't know which credential is being verified because we retrieve the entire list. 

The list can also be published in different places for redundancy, and also to make it harder for the issuer to monitor access to the list. The list could, for example, be published to a cdn.

The DCC advocates using a subtle variation to a simple list called a [Bitstring Status List](https://www.w3.org/TR/vc-bitstring-status-list/). Instead of simply adding ids to a list, we create a list in advance, where the list is a [bitstring](https://en.wikipedia.org/wiki/Bit_array) - effectively a string of zeros and ones, like so:

```00000000100000010100```

Each character position in that string is assigned to a single credential when the credential is issued. If the character at that position is a zero then the credential is active (hasn't been revoked). If the charcter is a one then the credential has been revoked.

So a verifier checking a credential has to download the right bitsting and then check the position in the string that has been allocated to the credential being checked.

The DCC provides two implemenations of a bistring - one that stores the bitstring in github and one that stores the bistring in a mongo db. Both implementations are packaged as a microservice that can be wired into a credential flow. We talk more about this service later.

### Public Key Authenticity - Registries

A fundamental problem with all cryptographically signed credentials is knowing who signed the credential. 

Anyone can easily generate a new key pair and sign a credential, so how do we know who generated (and used) the key pair that signed a given credential? I could, for example, create a Verifiable Credential that says I graduated from Oxford and sign it with a keypair I've just generated. A verifier would confirm that the signature was valid - that nothing in the credential had been tampered with - and that the credential had been signed by the private key associated with the indicated public key. How then does the verifier know that it wasn't Oxford who signed it?

The answer is a registry of public keys.

Oxford would add the public key part of their keypair to a well known registry - which is fundamentally just a list of public keys. The verifier would check the registry to confirm that the public key associated with a given credential was in the registry.

The registry of course has to be very carefully managed so that fake keys can't be added. Any new submissions or changes must be carefully vetted. These lists are public, though, and so subject to public scrutiny. It is not hard to imagine then that each institution would regularly confirm that only it's own legitimage keys are registred in it's name. Nevertheless, we do have to be careful about adding new keys.

Note that a signed credential can't itself point at the registry because then a fake credential could simply point at a fake registry. The verifier has to know which registires are legitimate - independently of anything in the credential.

Registries are still in their early days. It has yet to be worked out who will maintain registries, although a likely guess is that entities that already manage membership or accredation will manage the keys for its membership. 

For now the DCC has set of registries to which you can your keys:

[LIST them here]

Submit a pull request with your institutions key an we will review the submission.

Note too that if you have your own verifier - say a page on your web site at which your own issued verifiable credentials can be verifier (like say certificates your school has issued) then you will know which are your own public keys - you effectively are maintaining your own registry. You still have to be careful to build that check into your verification though - be sure to check that the public key associated with a credential is in fact one of your own.

We've talked about public keys here, but more likely is that you'll use a [Decentralized Identifier (DID)](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjTnLn24LeEAxU1C3kGHSTVAngQFnoECBMQAQ&url=https%3A%2F%2Fwww.w3.org%2FTR%2Fdid-core%2F&usg=AOvVaw2lmH5NVEyOOCzXxjwUG8Yh&opi=89978449) to manage your issuing keys, and at the moment, likely a did:web DID, which helps manage key rotation. The upshot is simply that the verifier confirms that a given DID is in the registry, rather than the public key. The verifier still, though, has to confirm that the given DID also manages the actual public key involved.

### Private Key Security

Your private key - the 'half' of a keypair that signs things - must be kept as secret as possible. Ideally you in fact don't ever want a human to see it. The risk is that if someone copies your signing key they can then issue credentials that will verify as authentic. And unless you happen across credentials that were fraudulently issued, you may well never know that your key has been compromised or that fraudulent credentials were issued with it.

There are a few options for securing your signing keys from least to most secure. Which you choose will likely depend on the type of credential you are issuing and institutional policy. For testing and evaluation, using an environment variable is the easiest choice, and how the DCC signing-service is configured by default.

#### Environment variable

Have someone manually generate the key and set it in an environment variable.

#### In-memory

Have your system code automatically generate a new key pair when the system starts or restarts, and possibly also at some regular interval like once a month.

The private part of the key pair will exist only in system memory and will never be written out anyhow. When the system shuts down, or the key is deliberately replaced, the old private key permanently disappears.

Your code will also have to publish (to a [registry](#public-key-authenticity-registries)) the new public key everytime a new key pair is generated.

#### Hardware Security Module

At the moment, the most secure option is the use a [Hardware Security Module (HSM)](https://en.wikipedia.org/wiki/Hardware_security_module). The HSM generates and manages keys, and signs things for you when needed.

An HSM does incur a cost, either for the hardware if you buy one to use in-house, or often per transaction if someone else manages it for you.

Whether to get an HSM often will depend on how concerned you are about fraud. You may be comfortable simply keeping your 

#### Mitigation

There are a few things you can do to mitigte the risk of a stolen key:

##### Rotate keys frequently

By switching your key you minimize the period in which a stolen key can be used. After the key has been "rotated out" (retired, decommissioned) the key can no longer sign new credentials.
This will only work, though, if you have a trustworthy timestamp attached to your credentials.

##### Expire credentials aggressively

This also only works if you have a trustworthy timestamp attached to the credential.

##### Public hash registry

Mention how a public hash registry can detect fraud????

### Wallets

### Verification

### Credential Display

### DCC components and services

The DCC has split out issuance into a number of components that can be assembled and configured to suit the needs of individual projects.

Our main components are:

The learner credential wallet (LCW)

Issuer coordinator

Exchange coordinator

signing-service

status-service

transaction-service

verifier-service

### Step by step guide

As a flow chart???

Do you have an existing datastore with credential data?
	- Would you like to tie directly into this store so that when recipients collect credentials, the credentials are dynamically generated using data from this store?

Do you want to issue credentials to the DCC Learner Credential Wallet?

Do you want to add a holder binding to your credentials. (A holder binding allows the holder to later prove they 'control' the credentials, in some sense like having a password)

Do you want to be able to revoke your credentials?
	- Are you comfortable using github to manage credential status?
	- Would you prefer to use your own mongo instance to manage credential status?

Do you want a stand-alone credential management system to which you add data (as a CSV) for credentials that can be collected by the recipients?

Do you want to notify recipients by email that they can collect a credential?

Do you want credential holders to be able to dynamically collect credentials from their institutional accounts?

Do you want login enforced when credentials are collected, or can they be collected using a token sent out in an email?


