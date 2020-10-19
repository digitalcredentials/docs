# Display Artifacts

This document describes how display artifacts (images, PDFs) are typically handled with DCC credentials. DCC credential authoring guidance is not prescriptive; these are just guidelines based on our practices.

## Types of display artifacts
We'll consider two categories of common display artifacts, which are the focus of this document:
1. supporting artifacts
2. alternate visual representations

### Supporting artifacts
This group encompasses supporting visual elements of a certificate, if desired -- such as an issuer logo. 

We generally handle such artifacts by embedding a data URI/URL (base64 encoded version) or linking to the artifact. There are important implementation considerations, which are discussed at length below. 

**Recommendation**: embed as data URI or link to a (possibly local) file

### Alternate visual representations
The typical use case for alternate visual representations is if the issuer intends to provide a human-readable display, whether for ceremonial purposes or intended for human verifiers. A typical example is if the issuer wants to issue a hybrid credential containing machine-readable data, but also indicating a visual form that represents the data, like a PDF. 

Our general recommendation is to link to the artifact. Further details about how this is accomplished in DCC credentials is below.

**Recommendation**: Link to the (possibly local) artifact

## Implementation Considerations

### Embedding a data URI/URL

Any content fully embedded in the credential has the advantage that it is part of the cryptographically signed content, and therefore has tamper-evidence. So content integrity is handled by default in this case.

There are many software libraries and tools supporting conversion of common image types to data URIs for embedding into the credential. 

Important size considerations:
- It's important to note that some tools/libraries impose URL length limitations, which may impact data URLs in non-standard uses. For example, certain browsers impose URL size limits of 2k, and such browsers would not allow a user (or software agent) to input the data URL into the browser URL field. However, this doesn't apply to DCC toolsets. It is up to the implementor to determine if such non-standard uses may be relevant. 
- At the same time, embedding too large a data URL can make it difficult for users that want to open the file in a json file viewer. We don't recommend any strict cutoff, above which an artifact should be linked to; this is a design consideration specific to the credential, and DCC reference examples demonstrate the different techniques that can be used.

> Special case: the ILR wrapper proposal uses a custom method of embedding data into a credential without using data URIs. Such techniques are supported in the DCC ecosystem as a Verifiable Credential, even if there's not richer toolset understanding of the embedded payload. (i.e. the DCC wallet may not inspect into the base64 content, but it can still store the credential and exchange it with other systems)

### Linked data

Linked data may be web-hosted or a local file.

For linked data there may be two additional considerations:
- Availability
- Modification 

**Availability**
If linking to web-hosted data, issuers must ensure the data will be permanently available at the designated URL. If linking to a local file, availability is not a problem. In this case, we recommend using the credential archive format to bundle the files, in addition to the guidance below.

**Modification**
A relying party must assume that any linked data may be modified. In some cases, this is acceptable (e.g. if the content is expected to be updated). In other cases, modification isn't desirable -- for example, if the issuer wants to sign the version of the content at that point in time. In the latter case, the issuer will want to use a uri with tamper-evidence.

Note that this is a special case of [referencing with/without integrity protection](https://www.w3.org/TR/vc-imp-guide/#referencing-other-credentials) from the VC implementation guide.


#### Linked data w/out tamper-evidence
As described in the VC implementation guidelines for [referencing credential content without integrity protection](https://www.w3.org/TR/vc-imp-guide/#referencing-credentials-without-integrity-protection), this is achieved by simply including a link (URI) to the target or including its well-known ID. This is recommended if the issuer wants the target data (referenced by the url) to be able to change.

#### Linked data w/ tamper-evidence

Otherwise, linked data can also be represented as a [hashlink](https://github.com/digitalbazaar/hashlink), which is a link with a content integrity check built in. 

There are three forms. Implementors will typically want to use the query parameter form for compatibility with existing systems

- Regular Hashlink (without URL encoded): `hl:zm9YZpCjPLPJ4Epc`
- Regular Hashlink (with URL encoded): `hl:zm9YZpCjPLPJ4Epc:z3TSgXTuaHxY2tsArhUreJ4ixgw9NW7DYuQ9QTPQyLHy`
- Hashlink as a query parameter: `https://example.com/hw.txt?hl=zm9YZpCjPLPJ4Epc`




