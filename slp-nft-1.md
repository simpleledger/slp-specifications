![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# NFT1 Specification
#### Specification version: 0.1
#### Date published: July 11, 2019



### Authors
Jonald Fyookball, James Cramer

### Acknowledgements
im_uname, Mark B. Lundeberg, Dexx7 for review and suggestions




## Simple NFT vs NFT1 Protocol

The [SLP Token Type 1 Protocol Specification](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md) was originally designed for fungible tokens.  It is possible to use the token type 1 protocol to support non-fungible tokens (NFT) by simply minting a non-divisible token supply of 1 without a minting baton, we call this a **simple NFT**.  However, this specification codifies a more capable type of NFT that allows grouping of many NFTs together, called **NFT1**.  

NFT1 is a simple extension to the SLP token type 1 protocol which allows many NFT tokens to be grouped together using a single ID.  Having the ability to group NFTs in a provable manner opens the doors for many more token applications, and makes SLP more similar to other NFT protocols (e.g., ERC-721).  NFT1 uses the same validation rules as SLP token type 1 with a few additional constraints and requirements described below.

## NFT1 Creation Procedure

Here is the summarized procedure for how to create a *grouped NFT* using this NFT1 protocol:

1. Create a new SLP token for **NFT1 group minting** (see specific requirements below).  The token ID of this new **NFT group minting** token will be considered to be the Group ID for NFTs that will be subsequently created.
    *   It is recommended that the decimal precision be set to 0 for the NFT group, but this is not required.
2. Create a new SLP token for a single **NFT child** (see specific requirements below) by spending a quantity  greater than 0 NFT1 group minting tokens in a new GENESIS transaction.
    *   Just prior to this step it is recommended that the wallet automatically breaks the Genesis' output quantity into multiple outputs using a fan-out SEND transaction, this will allow multiple NFTs to be created.  It is recommended that outputs that are intended to be used in to create NFT children have a value of 1, but this is not required.
3. Repeat Step 2 as many times as desired in order to create more NFT1 children that are part of the same  group.

## NFT1 Protocol Requirements

The following list of requirements are simple modifications or additions to a normal SLP Token Type 1 protocol requirements:

* NFT1 group minting token:
    * Token Type field set to `0x81` in all NFT1 parent GENESIS, MINT, and SEND transactions.
* NFT1 child tokens:
    * Token Type field set to `0x41` in all NFT1 child GENESIS and SEND transactions
    * GENESIS must include spending a valid NFT1 parent token (quantity > 0) at transaction input index 0.
    * GENESIS quantity shall be set to 1.
    * Shall not have a baton vout (baton vout must be set to `0x4c00` in NFT1 child GENESIS).
    * Shall have 0 decimal places (decimals must be set to `0x00` in NFT1 child GENESIS).

## Requirements for Wallets that only have SLP SEND

Wallets must ensure that NFT1 tokens are spent using the same Token Type field.  Spending an NFT1 group or child token with the wrong Token Type field in a SEND transaction will result in the token being burned.  NFT1 group minting tokens will be received as token type `0x81` and shall be spent as type `0x81`.  Likewise, NFT1 child tokens will be received as token type `0x41` and shall be spent as token type `0x41`.

## User Interface Recommendations

1. SLP wallets should present the total quantity of NFT1 child tokens being held by a user as a single line item *for each Group ID* in a list of all SLP tokens, and that line item should be expandable to show the list of individual NFT1 tokens within the NFT1 group.

2. SLP wallets should present the total quantity of NFT1 group minting tokens being held by a wallet in way that clearly distinguishes the Group quantity from the NFT child quantity (these are two separate line items).  Using a visual cues and nomenclature similar to the "minting baton" may help distinguish between NFT1 group minting supply and the NFT1 children that are part of the same group ID.

3. NFT1 child tokens can have different names, tickers, document URI, and document hashes from their respective NFT1 group minting token.  It will be up to the individual wallets to determine how to best present this information to its users.

4. NFT1 group token GENESIS/MINT GUIs should make it clear that the quantity to be minted is associated with the maximum number of NFT1 children that can be created.

5. NFT1 group token GENESIS/MINT GUIs should have the decimals field disabled and set to 0 by default, even though this is not a strict requirement of the protocol.
