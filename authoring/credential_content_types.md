# Credential Content Types

DCC advocates for credentials with machine-readable, semantically-rich content, enabled by linked data with published/interoperable contexts. However, well-known, interoperable contexts, schemas, and vocabularies are not yet widely available. Further, some approaches are using VCs as wrappers, i.e. metadata around a base64 encoded payload, as with ILR wrappers. 

DCC samples include examples of these range of credential types. Roughly, those categories are (TODO: link to examples of each when available):
- Vocabulary/taxonomy known and LD context is available
    - However, additional unmapped fields may be desired
- Machine-readable content available (e.g. merge fields into a template) but not yet mapped to an existing vocabulary
    - Need to create an LD context. Ideally, over time there will be convergence on standards
- Credential is thin wrapper around other content, not necessarily machine-readable
    - Example: ILR wrapper design, link to PDF with content-integrity

Notes:
- In all cases, any unmapped fields will need to be defined in an LD context -- hosted or inline.
- Related discussion: https://github.com/w3c-ccg/vc-examples/issues/31
