![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# Implementing Non-Fungible Tokens Using Simple Ledger Protocol
 

## Authors
Jonald Fyookball

## Acknowledgements
Dexx7, James Cramer, Mark B. Lundeberg for review and suggestions

## Extensions
Groupable, supply limitable NFT im_uname 20190331

# NFT
 
The [SLP Token Type 1 Protocol Specification](https://github.com/simpleledger/slp-specifications/blob/master/slp-token-type-1.md) was originally designed for fungible tokens.  However, it is possible to use the same protocol to support non-fungible tokens (NFT) as well.

Fungibility refers to the idea that all tokens are the essentially same.  For example, if there is a token representing credits on a gaming platform, all credits would be as good as any other credit.  Non-fungible tokens are those in which each token represents something unique.  The archetypal example is that of cryptokitties.

## Implementation Philosophy

Cryptokiities is based on Ethereum’s ERC-721 standard, which uses off-chain data for specifying the attributes of its tokens.  We can implement the same idea for NFT using the existing SLP protocol.

We considered the possibility of implementing NFT in a more on-chain manner, but the challenges of that approach are readily apparent.  Each NFT token class (such as cryptokitties or cryptopuppies etc) could desire its own data structure.  The validation of the minting, transfer, and mutation of any token could have unique rules and would thus require specialized nodes.  

That type of system  would be more compatible with a full-node token protocol like Wormhole.  It’s possible to implement a rule engine on a lite-wallet and combine it with checkpoint validation, but this seems to go against the SLP principle of simplicity. 

There may be some benefits to having a non-fungible token hold certain characteristics that are secured by the blockchain, and that can be explored at a future time.  For now, there is no immediate use-case, and it would rely on specialized validation anyway (which is likely to be centralized).  Therefore it is better to explore the options for using SLP as it exists today with an off-chain approach to metadata as is done in ERC-721.

## Multiple tokens per SLP genesis vs single token 

An SLP genesis transaction can mint any number of tokens.  One possible approach is to mint multiple tokens at once.  For example, if we wish to mint 100 cryptokitties, that could be done on a single transaction, which creates a single UTXO holding 100 tokens.  The tokens could then be dispersed as needed.

Besides only requiring 1 initial transaction, the advantage to this approach is that all tokens of the same class would share the same genesis id.  However, the disadvantage is that it complicates tracking of the tokens.  In this scheme, the unique identifier of a token would be the UTXO outpoint, which keeps changing as the token is moved around on the blockchain.  If a metadata system has to deal with a constantly changing id for each token, then this is a technical burden.

A better approach is to simply have one genesis transaction for each unique token.  This would give every individual token its own token id, which would not change.  Since the SLP token id is passed along with every token transfer, the id would be persistent and could be used by any off-chain system to quickly retrieve the metadata associated with the token.  The existing SLP validation engines can be used to ensure the integrity of the token chain.

The necessity to initiate an individual genesis transaction for every token is only a minor drawback since a transaction would eventually be needed to separate the token from the others in a set.  The main differences are that you would be unable to mint them en masse to set them aside, nor would you be able to limit the supply of any given token class.  The latter conern would certainly be a hinderence for fungible tokens, but is much less so for NFT.

Rather than adding additional metadata to the SLP genesis transaction to prove a token belongs to a certain group (such as a cryptographic signature), we can merely use the same Bitcoin address for issuance by convention.  In other words, all tokens in a class would have a genesis transaction coming from the same address.

## Extension: Groupable, supply limitable NFT tokens as a derivative of fungible tokens

As described above, a problem with the "one genesis per token" approach is that one cannot limit the supply for any token class; additionally, passing batons to another minter is also impossible as the tokens are strictly linked by genesis address. Moreover, this approach centralizes minting authority in the hands of a single keypair, and can limit the usefulness of the product as a result. An approach that takes advantage of existing fungible SLP tokens is described below. This approach shall result in NFT tokens with additional flexibilities, and whose metadata still exist entirely onchain just as the fungible tokens.

### Deriving NFT tokens from fungible tokens

Create a fungible token using a GENESIS transaction; for optimal NFT quantity management, it is recommended that the fungible token to be created without decimal places. In addition to the usual fields, add "NFT1 " at the beginning of document URL to specify that this token can be used to mint NFT tokens. 

To forge an NFT token, first prepare an UTXO with one or more of the NFT-enabled fungible tokens associated with it; then spend that UTXO in a GENESIS transaction that generates a new token with quantity of 1 and decimal places 0, prefixing its document URL with "NFT1_" followed by token hash of the fungible token for quick lookup. This simultaneously generates a new token and burn one fungible token. Wallets that support this scheme can check whether a given NFT token is valid by tracing DAG up to genesis, check if the tx burned any of the fungible token it specified, and check the validity of that back to the genesis of said fungible token. Each NFT token generated this way will have its own secondary "NFT id" from the txid of its genesis, and can remain distinct from each other.

The NFT token need not be absolutely limited in quantity by genesis of the fungible token; just like any other fungible tokens, the "generating" token can have its minting baton passed to the next address, maintaining flexibility if so wished.

### Maintaining control over minting

The scheme described above distributes NFT generating capability to any holder of the fungible token; it might be desirable for the minting authority to strictly control the ability to emit NFT tokens, but at the same time preserve guarantees about quantity limitations. To achieve that, simply generate the initial fungible tokens with the "NFT2 " prefix in document URL, and prefix NFT tokens generated with "NFT2_" followed by the fungible token's token id as above. The key difference is that NFT2 tokens can only originate from burning a fungible token held by the address that receives token from the genesis transaction; wallets should interpret a generation from UTXOs associated with any other address as invalid.

An exception to this rule is that if the minting baton is passed, then any address that received fungible tokens emitted by the baton also becomes valid generation addresses.

## Summary

SLP already allows NFT simply by issuing a unique token type for each token to be created.  Tokens can be grouped together into classes by following the guideline of issuing all tokens in a class from the same Bitcoin address.  Token issuers can create and modify metadata specific to each unique token and store this off-chain, using the token id as an immutable unique identifier. 

By linking genesis transactions to an existing fungible token, it is possible to create NFTs whose group and supply limits are easy to verify, similar to fungible tokens.
