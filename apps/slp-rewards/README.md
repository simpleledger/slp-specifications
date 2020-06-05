# SLP Rewards Standardization - DRAFT
A auditable token reward system for your customers

Last updated: June 5, 2020

## Purpose

Customer rewards are a common requirement for token issuers, and this is often the primary reason for issuing an SLP token.  Issuers and customers need a proper audit trail for documenting the history of sending rewards to their customers.  A strong audit trail boosts customer confidence and also satisfies bookkeeping requirements that either the token issuer or customer may have.

## Specification

Token rewards for a given token should be tracked using specified NFT1 Group ID provided by the token issuer.  Each token reward txid sent to token holders should be documented within the Document Hash field of an NFT Genesis transaction.  This will allow rewards that were in the form of either BCH or any SLP token to be tracked in the same way.

Token issuers can simply signal that an NFT1 Group token is for the purpose of reward auditing by creating a new NFT1 Group token with a Genesis having:

* ticker: "REWARDS"
* documentHash: "token id of token holders"

