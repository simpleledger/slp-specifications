![Simple Ledger Protocol](images/SLP-logo-solid-200.png)



# Token Document Protocol

#### Version: 0.1
#### Date published: TBD

## Purpose

A token can have arbitrary information attached using the `Document URI` field within the [GENESIS transaction](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md#genesis---token-genesis-transaction).  The purpose of this specification is to:

1. define a document schema to allow wallets, exchanges, and SDKs using SLP to maximize the user experience for SLP tokens (e.g., custom token icons, token author contact info, etc.), and

2. define a trustless protocol for publishing and updating a token document.

## Specification

### Version 1 Schema

The following pseudo schema provides the minimum format requirements for a token document to be considered valid. This schema is formalized in the `schemas/token-document-v1.schema.json` document.  

```js
{
    "v": 1,                             // (required) the token document schema version
    "icon":  "<svg> ... </svg>",        // (required) an SVG icon
    "auth": [ "simpleledger:..." ],     // (required) addresses allowed to replace this document
    "contact":                          // (required) contact info object
    {                              
        "email": "",                    // (required) email for support
        "phone": "",                    // (required) phone number for support
        "url": "",                      // (required) url for additional token details
    }
    "...": {                            // (optional) any additional app specific data
        "..."                         
    }
}
```

### Initial Publishing

The publishing of a token document shall meet the following requirements in order to be considered valid.

`token.json` must be: 

1. valid using `schemas/token-document-v1.schema.json`.
2. created prior to the GENESIS transaction, and referenced in the GENESIS `Document URI` and `Document Hash` fields.
3. uploaded to the Bitcoin Cash blockchain using the [Bitcoin Files Protocol](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md) with data stored on-chain, having `filename_utf8` set to `token.json` and `file_sha256_bytes` set to the valid hash of the uploaded content.

### Updating the Document

The token document may be superceded any number of times.  The procedure for updating the token document shall meet the following requirements in order to be considered a valid token document replacement.

`token.json` must be:

1. valid against the `schemas/token-document-v1.schema.json` valdation document
2. uploaded to the Bitcoin Cash blockchain using the [Bitcoin Files Protocol](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md) with data stored on-chain, having `filename_utf8` set to `token.json`, `file_sha256_bytes` set to the valid hash of the `token.json` uploaded content, and `previous_file_sha256_bytes` set to the bitcoin files protocol location (i.e, the file's location is the BFP metadata transaction ID) of the previous token document that is being superceded.
3. Uploaded the by one of the addresses listed in the `auth ` property of the the most recent `token.json` document
4. The first attempt to update a token document will be considered valid, subsequent attempts to update the same document will be ignored and considered invalid.  If the same document is updated two or more times within the same block then canonical transaction ordering will be used to identify which was the first attempt to update.

### Trust & Certification

Token wallets and block explorers want guard users against malicious content.  For this topic see [Token Trust Protocol](https://github.com/simpleledger/slp-specifications/blob/token-documents/slp-token-trust.md) specification.