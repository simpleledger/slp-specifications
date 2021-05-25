# SLP Staking Standard - DRAFT
*Standardized staking for your SLP token*

Last updated: June 8, 2020

## Purpose

Token staking is a common component and requirement for decentralized governance, reward mechanics, and coin transfer restrictions.  This specification provides the current methodology of staking that will be supported by the SLP ecosystem.  Of course other staking methods are possible, however, they will not likely be widely supported across the ecosystem if they are not listed in this peer-reviewed specification.

## Specification

The recommended method for token staking is to utilize a stateful covenant contract.  Validation of the staking contract addresses can be easily performed by examining the input scriptSigs of transactions containing p2sh outputs.

### Standard Staking Address

The following redeem script should be used to handle token staking:

`<insert the stateful covenant script contract here>`

The above stateful covenant locking script has two sequencial stages: (1) prepare and (2)  staking.  The following process describes the purpose of each state:

- (1) prepare: When coins are initially sent to this covenant contract they will be in the "prepare" state.  There are a couple of purposes for this initial stage.  
  - The first purpose of having this stage reduces the risk of losing tokens due to user error since the staking-contract is documented and the user's public key is known.  
  - The second purpose of this stage is that it provides on-chain documentation of what script the coins are being staked with after the coins have been moved to the staking stage.  
  - Under normal circumstances, a token holder would simply spend the tokens from the "prepare" stage address with the desired staking locktime value encoded as pushdata within the scriptSig.  The receiving address would be an address derived from the locktime and enforced by the covenant.  
  - Alternatively, while a coin resides at the prepare stage address, the user has the option to abort the staking position and return each token UTXO to its originating address.

- (2) staking: Once coins are sent to this address from the previous stage they will be locked for the timelock duration provided by the token holder in the scriptSig.  Anyone will be able to verify the coins are staked since the covenant script template was published in the upstream transaction along with the user specified staking timelock.

### Issuer Multisig Staking Address

In the future token issuers may want to have multisig capabilities for signing on the issuer side, at this time no recommendations have been developed for this use case.  Due to the interactive process of multisig, additional security concerns related to the birthday attack problem may arise and should be mitigated carefully.

### P2SH Verification Procedure

A p2sh address must be verified as being an address created from the hash160 of a time-locked staking contract before the issuer will treat it as staked coins.

1) Query for all p2sh address type holders of a token

2) For each transaction associated with a p2sh holder, enumerate the token input scriptSigs.

3) For each input scriptSig, attempt to compute a p2sh address using the above staking contract templates using the public keys found within the script sigs.

4) If any of the computed p2sh addresses match the output holding p2sh, then you have verified the current p2sh holding tokens is indeed a staking address.