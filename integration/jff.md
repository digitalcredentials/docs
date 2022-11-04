# DCC JFF Plugfest 2 Integration Guide

A quick guide for JFF partners, who would like to integrate with either the [DCC Issuer (sign-and-verify)](https://github.com/digitalcredentials/sign-and-verify) or the [DCC Learner Credential Wallet (LCW)](https://github.com/digitalcredentials/learner-credential-wallet)

## Table of Contents

- [DCC JFF Plugfest 2 Integration Guide](#dcc-jff-plugfest-2-integration-guide)
  * [JFF Plugfest 2 Overview](#jff-plugfest-2-overview)
  * [Integrating with the DCC Issuer](#integrating-with-the-dcc-issuer)
  * [Integrating with the DCC Learner Credential Wallet](#integrating-with-the-dcc-learner-credential-wallet)
    + [Overall Flow](#overall-flow)
    + [Installing the Wallet](#installing-the-wallet)
    + [Constructing the Deep Link](#constructing-the-deep-link)
    + [Sharing the Deep Link](#sharing-the-deep-link)
    + [Add your Issuing DID to the DCC Registry](#add-your-issuing-did-to-the-dcc-registry)


## JFF Plugfest 2 Overview

More details about the second round of the JFF Plugfest:

[JFF Interoperability Plugfest 2](https://w3c-ccg.github.io/vc-ed/plugfest-2-2022/)

## Integrating with the DCC Issuer

The DCC Isssuer is called [sign-and-verify](https://github.com/digitalcredentials/sign-and-verify). It is a NodeJS server for issuing verifiable credentials, implementing a subset of the [VC-API specification](https://w3c-ccg.github.io/vc-api).

To enable easier integration with JFF partner wallets, the DCC sign-and-verify issuer will be registered with the [chapi.io playground](https://playground.chapi.io/issuer)

To import a DCC signed JFF PF2 Open Badge into your wallet, open the [playground](https://playground.chapi.io), and:

1. From the options screen (click the little wheel to the top right):

- choose DCC as the issuer
- enable DIDAuth

<img width="1031" alt="image" src="https://user-images.githubusercontent.com/547165/199337511-6660eee9-6645-4717-9ca5-72bdd216cbc3.png">

2. Click the 'JFF x vc-edu PF2' button to select it as the credential type.

<img width="1030" alt="image" src="https://user-images.githubusercontent.com/547165/199337806-bffb7030-bb72-4406-8e3a-0596d5a3521f.png">

3. Click 'Authenticate and Generate VC' to request your credential.  The playground will guide you through the process of choosing your wallet and importing the credential.

<img width="1030" alt="image" src="https://user-images.githubusercontent.com/547165/199338229-fe0d8329-32c0-41f7-b58a-c27f038f2df2.png">

Coming soon:  link to CHAPI docs that explain how wallets interact with the playground.

## Integrating with the DCC Learner Credential Wallet

For JFF 2, the Learner Credential Wallet (LCW) will be invoked with a deep link like this example:

`org.dcconsortium://request?auth_type=bearer&issuer=jff&challenge=90u09j04&vc_request_url=http://issuer.myserver.org/exchange/8989844`

### Installing the Wallet

IMPORTANT:  For the JFF PlugFest *DO NOT* use the version of the wallet that is in the Apple App Store or the Google Play Store (or linked from [lcw.app](https://lcw.app/)).

You should instead either:

- build locally from the [LCW github repository](https://github.com/digitalcredentials/learner-credential-wallet)

OR
- contact us to get a TestFlight account

Once you have a local build running or a TestFlight instance, then:

- Setup the Learner Credential Wallet by selecting ‘Quick Setup (Recommended)’ and creating a passphrase to secure the wallet.
- Select the deep link from a web page on your phone

### Overall Flow

![](wallet-server-flow.png)

### Constructing the Deep Link

The deep link must start with either:

- 'dccrequest://request?'
- 'org.dcconsortium://request?'

and must include these four request parameters, with the values as specified:

- <b><i>auth_type=bearer</i></b>

- <b><i>issuer=jff</i></b>

- <b><i>challenge={your_generated_value}</i></b>

The wallet will add the challenge value to a DID Auth Verifiable Presentation that it signs with the holder's DID, so as to foil replay attacks.  It is therefore up to you (the issuer) to pass in a challenge that you can later verify when the VP is submitted to your vc_request_url.  You might also want to use the challenge for other reasons, for example, to identify the specific credential being requested, or to act as a kind of bearer token that confirms that the end user had authenticated in some prior step.  There is, however, also a 'challengeId' path parameter on the vc_request_url (defined below) that you can use to for similar things, like identifying the particular credential instance.

- <b><i>vc_request_url={your_issuer_url}</i></b>

This is your endpoint from which the wallet will request the credential.  This is called /exchange/{exchangeId} in the VC-API spec.  So, an example might be: https://myissuer.org/exchange/8989844, but it is of course entirely up to you how your url is named since the wallet simply takes it as-is and invokes it, passing in a DID Auth with the wallet holder’s DID, and with the challenge described below.  Note that the 'exchangeId' can be anything you like, but is intended to identify the specific instance of the credential being requested.


When invoked with this deeplink, the wallet will send a standard DID Auth VP to ’vc_request_url’ containing:

 - the holder DID to which to issue the credential
 - the 'challenge' that had been orginally passed in on the deep link.

Here is an example DID Auth (taken from https://w3c-ccg.github.io/vp-request-spec/#example-a-did-authentication-response):

```
{
  "@context": ["https://www.w3.org/2022/credentials/v2"],
  "type": "VerifiablePresentation",
  "holder": "did:example:12345",
  "proof": {
    "type": "DataIntegrityProof",
    "cryptosuite": "eddsa-2022",
    "verificationMethod": "did:example:12345#key-1",
    "challenge": "99612b24-63d9-11ea-b99f-4f66f3e4f81a",
    "domain": "example.com",
    "created": "2022-02-25T14:58:42Z",
    "proofPurpose": "authentication",
    "proofValue": "z3FXQjecWufY46...UAUL5n2Brbx"
  }
}
```

The wallet then expects to receive in return the Verifiable Credential for the holder, with the submitted holder DID included as the subjectId in the returned VC.

### Sharing the Deep Link

NOTE:  the following explanation probably isn't relevant for the JFF plugfest where the sharing will be part of the recorded demonstration of integration, but we include it here if it is of interest.

You can share the deeplink anyway you like, but the two most common approaches are to 
share it in a web page (say from a course page from which a student wants to collect 
a verifiable copy of their course completion) or in an email sent out to the student.

If you do want to share the link in an email, you should use an email-friendly variant of the
deep link that works better with email clients.  The variant simply
replaces the base uri (`org.dcconsortium://request?` or `dccrequest://request?`) with:

`https://lcw.app/request.html?`

So if your deeplink is:

`org.dcconsortium://request?auth_type=bearer&issuer=jff&challenge=90u09j04&vc_request_url=http://issuer.myserver.org/exchange/8989844`

then your email-friendly link will be:

`https://lcw.app/request.html?auth_type=bearer&issuer=jff&challenge=90u09j04&vc_request_url=http://issuer.myserver.org/exchange/8989844`

The link opens a web page which then redirects to the deeplink.  This slight redirection is needed because some email clients strip out the deeplinks.

### Add your Issuing DID to the DCC Registry

Finally, you'll need to 'register' your issuing DID with the DCC Community Registry.  The registry is simply a json file in a github repository:

[https://github.com/digitalcredentials/community-registry/registry.json](https://github.com/digitalcredentials/community-registry/blob/main/registry.json)

Add your entry to the registry file, following the examples already there, and submit it as a Pull Request to the repository.

And that’s it!
