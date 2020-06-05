# SLP Staking Standardization - DRAFT
Add a staking system to your SLP token

Last updated: June 5, 2020

## Purpose

Token staking is a common component and requirement for decentralized governance, reward mechanics, and coin transfer restrictions.  This specification provides the current methodology of staking that will be supported by the SLP ecosystem.  Of course other staking methods are possible, however, they will not likely be widely supported across the ecosystem if they are not listed in this peer-reviewed specification.

## Specification

Currently the method for handling customer token staking is to have customer tokens based on a pay-to-script-hash (P2SH) address that is generated from a customer supplied public key.  Validation of the staking contract addresses can be performed by examining the scriptSig of .

### Standard Staking Address

The following redeem script should be used to handle non-multisig instances where a token issuer requires each coin to be time locked for a certain period of time with no additional transfer restrictions:

`<locktime> OP_CHECKLOCKTIMEVERIFY OP_DROP <customer_pubkey> OP_CHECKSIG`

### Multisig Staking Address

The following redeem script should be used for situations when the token issuer needs to further restrict coin transfer beyond a time lock:

`<locktime> OP_CHECKLOCKTIMEVERIFY OP_DROP N <customer_pubkey> <issuer_pubkey1> ... <issuer_pubkeyN-1> N OP_CHECKMULTISIG`

For the purpose of simple p2sh address verification described below, only M-of-N multisig where M=N are recommended.

### P2SH Verification Procedure

A p2sh address must be verified as being created from the hash160 of the staking contract before the issuer will treat it as staked coins.

1) Query for all p2sh address type holders of a token

2) For each transaction associated with a p2sh holder, enumerate the token input scriptSigs.

3) For each input scriptSig, attempt compute a p2sh address using the above staking contract templates using the public keys found within the script sigs.  Taking care to distinguish between p2pkh and p2ms.

4) If any of the computed p2sh addresses match the output holding p2sh, then you can be sure the current p2sh holding tokens is indeed a staking address.
