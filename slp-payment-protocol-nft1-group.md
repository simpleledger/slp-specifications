![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# NFT1 Group Payment Protocol

#### Version: 0.1
#### Date published: March 3, 2020

## Purpose

This specification is similar to the Simple Ledger Payment Protocol except this protocol is for a specific edge case related to the [NFT1 specification](./slp-nft-1.md).  This protocol allows a merchant to express intent to accept any NFT token ID that has been created from a specific NFT1 Group ID.

In summary, the differences between this protocol and the Simple Ledger Payment Protocol are:
* The MIME types are named differently (i.e., "simpleledger-nft1-group-*")
* The SLP OP_RETURN metadata within the payment request contains a NFT1 group ID in place of the final token ID which shall be replaced by the wallet.
* The SLP OP_RETURN metadata within the payment has a token type of 0x41, this signals the merchant's final intent to accept and NFT1 child 

## Specification

### Extending Existing Payment Protocol

This payment protocol is an extension of [BIP 70](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki), [BIP 71](https://github.com/bitcoin/bips/blob/master/bip-0071.mediawiki), [BIP 72](https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki), and the [Simple Ledger Protocol URI Scheme Specification](https://github.com/simpleledger/slp-specifications/blob/token-documents/slp-uri-scheme.md). Applications which support these protocols are able, with minor modifications, to support the payment protocol described in this specification.

### Payment Request Generation

Payment Requests are generated according to the [BIP 70 specification](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki#PaymentDetailsPaymentRequest) with the following additional requirements:

1. Outputs must be in the proper order according to the Simple Ledger Token specification
2. The first output in the Payment Request (vout=0) must be the SLP OP_RETURN token type 0x41 output containing an NFT1 Group ID to signal the merchant's acceptance of any valid 0x41 token created from that group ID.
3. All outputs listed in the SLP OP_RETURN output must have a corresponding output in the Payment Request (vout=1-n)
4. The transaction defined by the Payment Request must conform to the specification for a [SEND transaction of SLP Token Type 1](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md#send---spend-transaction)

### Payment Generation

Payments are generated according to the [BIP 70 specification](https://github.com/bitcoin/bips/blob/master/bip-0070.mediawiki#Payment) with the following additional considerations:

1. Outputs in the transaction must be listed in the exact order prescribed by the Payment Request
2. Additional outputs, such as those for sending change, may be appended to the SLP OP_RETURN script at index 0

### Payment Validation

Payments are considered valid if the submitted transaction has the outputs required by the Payment Request. As with BIP 70, the script and amount for all required outputs, except the output at index 0, must match the details in the Payment Request. The SLP OP_RETURN script (vout=0) does not have to exactly match the script in the Payment Request to be considered valid. However, it must include any requested outputs.

#### Example OP_RETURN Script Validation

REQUESTED SCRIPT (ASM) - :</br>
``OP_RETURN 534c5000 41 53454e44 c01213b295920af4ca428212442817bb6a7b231aefd3f4b79c40fd3271cd5ea3 0000000000000001``

VALID SCRIPT (ASM) - APPENDED CHANGE OUTPUT:</br>
``OP_RETURN 534c5000 41 53454e44 617f449288e2d11791eb2ce03a96a03ec1604d133500030029b43fa8b3ea21ef 0000000000000001``

INVALID SCRIPT (ASM) - MISSING REQUIRED OUTPUT:</br>
``OP_RETURN 534c5000 41 53454e44 617f449288e2d11791eb2ce03a96a03ec1604d133500030029b43fa8b3ea21ef``

### URI Specification

The URI scheme follows [BIP 72](https://github.com/bitcoin/bips/blob/master/bip-0072.mediawiki), and the [Simple Ledger Protocol URI Scheme Specification](https://github.com/simpleledger/slp-specifications/blob/token-documents/slp-uri-scheme.md).

#### Examples

A backwards-compatible request:</br>
``simpleledger:qqmtw4c35mpv5rcjnnsrskpxvzajyq3f9ygldn8fj0?amount1=1-<xyzGroupID>-isgroup&label=Satoshi-Nakamoto&r=https://merchant.com/pay/3D2a8628fc2fbe``

Non-backwards-compatible equivalent:</br>
``simpleledger:?r=https://merchant.com/pay/3D2a8628fc2fbe``

### MIME Types

The MIME (RFC 2046) Media Types follow the basic specification described in [BIP 71](https://github.com/bitcoin/bips/blob/master/bip-0071.mediawiki). The Media Type (Content-Type in HTML/email headers) for messages shall be:

| Message  | Type/Subtype                                  |
| ------------ | ------------------------------------------|
| PaymentRequest | application/simpleledger-nft1-group-paymentrequest |
| Payment      | application/simpleledger-nft1-group-payment          |
| PaymentACK      | application/simpleledger-nft1-group-paymentack    |

#### Example

A web server generating a PaymentRequest message to initiate the payment protocol would precede the binary message data with the following headers:

``Content-Type: application/simpleledger-nft1-group-paymentrequest``</br>
``Accept: application/simpleledger-nft1-group-payment``</br>
``Content-Transfer-Encoding: binary``</br>

## Reference Implementations

### Clients
None currently

### Libraries
None currently
