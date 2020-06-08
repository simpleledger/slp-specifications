# SLP Rewards Standard - DRAFT
*A auditable token reward system for token issuers and holders*

Last updated: June 8, 2020

## Purpose

Rewards are a common requirement for token issuers, and this is often the primary reason for issuing an SLP token.  Issuers and token holders need a proper audit trail for documenting the history of sending rewards to token holders.  A strong audit trail boosts token holder confidence and also satisfies bookkeeping requirements that either the token issuer or holder may have.

## Specification

Token rewards for a given token should be tracked using specified NFT1 Group ID provided by the token issuer.  Each token reward TXID sent to token holders should be documented within the Document Hash field of an NFT Genesis transaction.  This will allow rewards in the form of BCH or any arbitrary SLP token to be tracked in the same way.

### Rewards Trail

Token issuers can simply signal that an NFT1 Group token is for the purpose of providing an auditable "rewards trail" by creating a new NFT1 Group token with a Genesis having:

* ticker: `__REWARDS_TRAIL` (the double underscore indicates this token is being used at an application level, not for typical interactions with users)
* documentHash: `<tokenId associated with this sending of rewards>`
* documentUrl: location of any additional information related to this 

For each rewards related transaction sent to token holders a new NFT token should be created with the following format:

* ticker: `__REWARD`
* documentHash: `<txid associated containing rewards sent to token holders>`
* documentUrl: location of any additional information related to the reward, to be used at the discretion of the token issuer.

### Considerations

* Token issuers may want to issue rewards for different purposes to the same token holders.  There is nothing wrong with using the same Rewards Trail for different purposes, however, having unique NFT1 Groups dedicated for each rewards purpose may offer an improved user experience.