![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# Simple Ledger Postage Protocol

#### Version: 0.2
#### Date published: November 3, 2019

## Purpose

This specification describes a protocol for the sender of SLP tokens to effectuate a transaction without the need for additional native Bitcoin Cash (BCH) inputs to cover output balance and/or miner fees. Use of this protocol enables wallets to exclusively support SLP tokens without also, simultaneously, having to act as a wallet for native BCH.

## Background

Simple Ledger Protocol (SLP) transactions must obey the same basic consensus rules as all other Bitcoin transactions. All inputs and outputs must be valid. The sum of inputs, in satoshis, must be greater than the sum of the outputs, with the remainder of sufficient value so as to pay the minimum mining fee. Unlike standard BCH transactions, the nominal value of a given SLP transaction (as denominated in the SLP token itself) is independent of the satoshi values used by a node to validate the transaction. By convention (and according to the Simple Ledger Protocol specification), each SLP input and output, regardless of SLP value, has a value, in satoshis, of 546. This necessarily means that unless the number of inputs in a given SLP transaction greatly outnumber the outputs, additional, non-SLP, BCH inputs must be included in an SLP transaction in order to ensure it is valid.

The additional non-SLP BCH inputs attached to most SLP transactions are akin to the concept of "gas" as used in Ethereum. In order to send a token from one party to another, on-chain, a wallet must also have some amount of the additional native token (in this case BCH satoshis) on hand. This limitation introduces significant friction to the user experience. An SLP token sent, for instance, to a fresh wallet cannot be utilized by the recipient unless he or she is also in possession of additional BCH. This means that a sender must either send some additional BCH along with the SLP tokens, or the recipient must acquire BCH through other means - a potentially daunting task for new users.

The protocol described in the following specification allows any SLP wallet to engage an intermediary (**"post office"**) who will add BCH inputs (**"postage"**) to the sender's transaction, and then broadcast it to the network. In exchange for providing this service, the post office can require payment, in the form of the SLP token, as an additional output in the transaction. This interaction effectively allows a sender to pay his network fees in SLP tokens as opposed to in BCH.


## Specification

### Extending Existing Payment Protocol

This payment protocol is an extension of a portion of the [Simple Ledger Payment Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md) and the [JSON Merchant Data](https://github.com/jeton-tech/payment-protocol-extensions/blob/master/json-merchant-data.md) specifications. Applications supporting these protocols are able, with minor modifications, to support the payment protocol described in this specification.


### Postage Rate Request

In order to determine the required postage rate for a given token, the sender's wallet queries the post office server. The server returns postage rate data which can then be used by the wallet to calculate the appropriate amount of SLP token value to include in an additional output as well as the proper payment address. Postage is paid in "stamp" units. Each stamp represents a certain number (typically 365) of satoshis that will be added to the transaction.

#### REQUEST
A GET request should be made to the Postage Protocol url.

#### Headers
* `Accept` should be set to `application/json`.

#### Response
The response will be a JSON format payload.

#### Response Body
* `version` - The version of the Postage Rate Request specification. This document defines version 1
* `address` - The address to which postage payment must be made
* `weight` - The number of satoshis represented by each postage unit (stamps)
* `transactionttl` - The time (in seconds) that utxos added as stamp inputs will be guaranteed valid in the case that the sender does not want the post office to broadcast the transaction to the Bitcoin Cash network
* `stamps` - An array of stamp objects, representing the tokens supported by the post office and the rate per stamp
    * `name` - The human readable name of the SLP token
    * `symbol` - The ticker symbol of the token
    * `tokenId` - The token ID hash for the token
    * `groupId` - The token ID hash associated with an [NFT1 group](./slp-nft-1.md). If `tokenId` and `groupId` fields are both provided `groupId` shall be ignored.  The `rate` associated with a specified group ID should be set to 0 since NFTs only have a quantity of 1 and SLP currently only allows the transfer of a single token ID at a time.
    * `decimals` - The number of decimal places for the token
    * `rate` - The rate in base units (before factoring for `decimals`)

#### Response Body Example
```
{
   "version":1,
   "address":"simpleledger:qrxj0mftnsrl63uqwn2jcsxvwymgxm7sev7dyx7hrr",
   "weight":365,
   "transactionttl":30,
   "stamps":[
      {
         "name":"Spice Token",
         "symbol":"SPICE",
         "tokenId":"4de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf",
         "decimals":8,
         "rate":727418066
      },
      {
         "name": "DND Swords and Shields",
         "symbol": "DND",
         "groupId": "c01213b295920af4ca428212442817bb6a7b231aefd3f4b79c40fd3271cd5ea3",
         "decimals": 0,
         "rate": 0
      },
      {
         "name":"Honest Coin",
         "symbol":"USDH",
         "tokenId":"c4b0d62156b3fa5c8f3436079b5394f7edc1bef5dc1cd2f9d0c4d46f82cca479",
         "decimals":2,
         "rate":1
      },
      {
         "name":"AnyPay Gold",
         "symbol":"GOLD",
         "tokenId":"8e635bcd1b97ad565b2fdf6b642e760762a386fe4df9e4961f2c13629221914f",
         "decimals":3,
         "rate":1
      }
   ]
}
```

#### Integration With Simple Ledger Payment Protocol

A merchant utilizing [Simple Ledger Payment Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md) to process payments can, optionally, act as a post office for buyers. In such a case, the Payment may be sent to the payment protocol server with an additional SLP output paying for postage. The merchant's server will add the required satoshis, after validating the payment and before broadcasting the transaction to the Bitcoin Cash network. A merchant wishing to offer this service will include the Postage Rate object in the Payment Request sent to the buyer. The Postage Rate object will be contained in the `postage` property of the [JSON Merchant Data](https://github.com/jeton-tech/payment-protocol-extensions/blob/master/json-merchant-data.md) of the Payment Request.

#### Payment Request JSON Merchant Data Object Example
```
{
   "postage":
      {
         "version":1,
         "address":"simpleledger:qrxj0mftnsrl63uqwn2jcsxvwymgxm7sev7dyx7hrr",
         "weight":365,
         "transactionttl":30,
         "stamps":[
            {
               "name":"Spice Token",
               "symbol":"SPICE",
               "tokenId":"4de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf",
               "decimals":8,
               "rate":727418066
            },
            {
               "name": "DND Swords and Shields",
               "symbol": "DND",
               "groupId": "c01213b295920af4ca428212442817bb6a7b231aefd3f4b79c40fd3271cd5ea3",
               "decimals": 0,
               "rate": 0
            },
            {
               "name":"Honest Coin",
               "symbol":"USDH",
               "tokenId":"c4b0d62156b3fa5c8f3436079b5394f7edc1bef5dc1cd2f9d0c4d46f82cca479",
               "decimals":2,
               "rate":1
            },
            {
               "name":"AnyPay Gold",
               "symbol":"GOLD",
               "tokenId":"8e635bcd1b97ad565b2fdf6b642e760762a386fe4df9e4961f2c13629221914f",
               "decimals":3,
               "rate":1
            }
         ]
      }
}
```


### Payment Generation

Payments are generated according to the Payment Generation section of the [Simple Ledger Payment Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md) with the following additional considerations:

1. Only valid Simple Ledger Protocol inputs and outputs may be included. No non-SLP inputs or outputs may be included in the transaction
2. All inputs must use the SIGHASH_ALL | SIGHASH_ANYONECANPAY signature hash type. This is to mitigate any potential risk of malleation of outputs.
3. If required by the post office, an additional output containing sufficient SLP token value to cover postage must be included in the transaction.
4. By default, the post office will broadcast the transaction to the Bitcoin Cash network once adequate postage has been added. If the sender does not wish for the transaction to be broadcast, and instead wishes for the valid raw transaction hex, with postage, to be returned in the Payment ACK, this must be specificed in a [JSON Merchant Data](https://github.com/jeton-tech/payment-protocol-extensions/blob/master/json-merchant-data.md) object in the Payment.

### Payment Validation

The post office will validate any submitted payment, ensuring that it is an otherwise valid SLP transaction, before appending additional outputs and broadcasting the transaction to the Bitcoin Cash network or returning the raw transaction hex to the sender in the PaymentACK.

### JSON Merchant Data Object

In certain cases, the sender of a transaction may not want the post office to broadcast the valid, postage-paid transaction to the Bitcoin Cash network. For instance, if the wallet is paying a Simple Ledger Payment Protocol request, the post office can add stamps and return the raw transaction hex to the wallet client in the Payment ACK, which can subsequently forward the transaction on to the Payment Protocol server. 

#### Example Return Postage Paid Transaction JSON Merchant Data Object

```
{
   "broadcastRawHex":false
}
```

If the raw transaction hex is returned to the sender, the post office will "quarantine" the UTXOs used as stamp inputs, making them unspendable for the time period specified in the `transactionttl` of the Postage Rate Request.

If a JSON Merchant Data object meeting the above specification is not included with the Payment, the postage paid transaction will be broadcast to the Bitcoin Cash network by the post office.


### Stamp Inputs

When the sender's transaction is determined to be valid, the post office appends one or more inputs to the transaction and signs those inputs.

1. The sum of the appended outputs must be sufficient to make the transaction valid and cover the minimum required miner fee.
2. All inputs will be signed with the SIGHASH_ALL signature hash type. The SIGHASH_ANYONECANPAY signature hash type will not be used on stamp inputs.

Because an additional change output cannot be added to the transaction, due to the sender signing inputs with SIGHASH_ALL, it is most efficient for post office providers to have an available stock of stamp UTXOs, of one or more standardized "weights," with the smallest UTXO value corresponding to the weight amount defined in the Postage Rate Request.

#### Example Transaction

BEFORE POSTAGE (Single SLP Input):</br>

| INDEX | INPUT | OUTPUT |
| ------------ | ------------ | ------------------------------------------|
| 0 | **(70 SLP tokens)** 546 satoshis  | **SLP OP_RETURN** |
| 1 | | **(20 SLP tokens)** 546 satoshis |
| 2 | | **(30 SLP tokens)** 546 satoshis  *paying postage* |
| 3 | | **(20 SLP tokens)** 546 satoshis  *change* |

AFTER POSTAGE (Single SLP Input):</br>

| INDEX | INPUT | OUTPUT |
| ------------ | ------------ | ------------------------------------------|
| 0 | **(70 SLP tokens)** 546 satoshis | **SLP OP_RETURN** |
| 1 | 546 satoshis *stamp* | **(20 SLP tokens)** 546 satoshis |
| 2 | 546 satoshis *stamp* | **(30 SLP tokens)** 546 satoshis  *paying postage* |
| 3 | 546 satoshis *stamp* | **(20 SLP tokens)** 546 satoshis  *change* |
| 4 | 546 satoshis *stamp* | |

BEFORE POSTAGE (Multiple SLP Inputs):</br>

| INDEX | INPUT | OUTPUT |
| ------------ | ------------ | ------------------------------------------|
| 0 | **(10 SLP tokens)** 546 satoshis | **SLP OP_RETURN** |
| 1 | **(35 SLP tokens)** 546 satoshis | **(20 SLP tokens)** 546 satoshis |
| 2 | **(25 SLP tokens)** 546 satoshis | **(30 SLP tokens)** 546 satoshis  *paying postage* |
| 3 | | **(20 SLP tokens)** 546 satoshis  *change* |

AFTER POSTAGE (Multiple SLP Inputs):</br>

| INDEX | INPUT | OUTPUT |
| ------------ | ------------ | ------------------------------------------|
| 0 | **(10 SLP tokens)** 546 satoshis | **SLP OP_RETURN** |
| 1 | **(35 SLP tokens)** 546 satoshis | **(20 SLP tokens)** 546 satoshis |
| 2 | **(25 SLP tokens)** 546 satoshis | **(30 SLP tokens)** 546 satoshis  *paying postage* |
| 3 | 546 satoshis *stamp* | **(20 SLP tokens)** 546 satoshis  *change* |
| 4 | 546 satoshis *stamp* | |

AFTER POSTAGE (Multiple SLP Inputs / Varying Stamp Weights):</br>

| INDEX | INPUT | OUTPUT |
| ------------ | ------------ | ------------------------------------------|
| 0 | **(10 SLP tokens)** 546 satoshis | **SLP OP_RETURN** |
| 1 | **(35 SLP tokens)** 546 satoshis | **(20 SLP tokens)** 546 satoshis |
| 2 | **(25 SLP tokens)** 546 satoshis | **(30 SLP tokens)** 546 satoshis  *paying postage* |
| 3 | 911 satoshis *stamp* | **(20 SLP tokens)** 546 satoshis  *change* |


### MIME Types

The MIME (RFC 2046) Media Types follow the basic specification described in [Simple Ledger Payment Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md)

| Message  | Type/Subtype                                  |
| ------------ | ------------------------------------------|
| Payment      | application/simpleledger-payment          |
| PaymentACK      | application/simpleledger-paymentack    |

#### Example

A wallet client generating a Payment message would precede the binary message data with the following headers:

``Content-Type: application/simpleledger-payment``</br>
``Accept: application/simpleledger-paymentack``</br>
``Content-Transfer-Encoding: binary``</br>


### Common Errors

| HTTP Status Code | Response | Cause |
|---|---|---|
| 400 | Unsupported Content-Type for payment | Your Content-Type header was not valid |
| 400 | Invalid Payment | The payment is would be invalid even if postage was added |
| 400 | Could not parse OP_RETURN output | SLP OP_RETURN output is malformed |
| 400 | Unsupported SLP token | The SLP token in the transaction is not among the supported tokens |
| 400 | Insufficient postage paid | The amount paid to the post office address is insufficient to cover postage |
| 400 | Stamps currently unavailable. In need of refill | The post office has insufficient stamp UTXOs available. Please inform post office support |
| 500 | Error | An error has occurred on the post office server |


### Notes On Privacy And Censorship

In a case where a post office server cannot determine the nature or source of a transaction, the sender's privacy is maintained. There is also little incentive for a post office to refuse any given transaction. However, in the case where a post office determines that a transaction benefits a (business) competitor, they may perceive greater benefit in not adding postage and refusing the transaction than would be received from the revenue gained by providing postage. To follow are some optional, but beneficial practices wallets can follow to keep incentives aligned between sender and post office.

1. Non-reuse of addresses by senders and third parties such as payment protocol servers. This prevents potential refusal of transactions based on benefit to business competitors.
2. Non-inclusion of [JSON Merchant Data](https://github.com/jeton-tech/payment-protocol-extensions/blob/master/json-merchant-data.md) from [Simple Ledger Payment Protocol](https://github.com/simpleledger/slp-specifications/blob/master/slp-payment-protocol.md) requests being fulfilled by a particular transaction. Although it may be simpler to include this data and merely append the `broadcastRawHex` property, this potentially reveals private information about the buyer and/or merchant. It should be assumed that a post office will retain data about its transactions, and inclusion of such sensitive information represents a risk.


### Notes On Transaction Malleability

When using the SIGHASH_ANYONECANPAY signature hash type with a transaction with SLP inputs, transaction malleability should be considered. In the case of a transaction with multiple SLP inputs, all using SIGHASH_ANYONECANPAY, replacing one of the inputs with an input which is invalid for the given SLP token can invalidate the SLP outputs and, if broadcast, burn any valid SLP input still included in the transaction. Post office providers have every incentive not to allow this to occur, since that would invalidate their own postage payment, which is an SLP output. Additionally, by using the standard SIGHASH_ALL (without SIGHASH_ANYONECANPAY) to sign the stamp inputs, the post office prevents any inputs from being replaced, including the SLP inputs using SIGHASH_ANYONECANPAY.

The post office still faces the potential risk of an unconfirmed parent of the sender's transaction being malleated. This would invalidate the postage paid transaction and the post office would not receive their SLP payment, though the stamp inputs would return to their state as UTXOs in control of the post office. To prevent such an occurence, the post office may, at their discretion, reject any sender transaction with an unconfirmed parent that might be susceptiple to malleation.


## Reference Implementations

### Clients
None Currently

### Libraries
None currently
