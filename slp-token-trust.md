![Simple Ledger Protocol](images/SLP-logo-solid-200.png)



# Token Trust Protocol

#### Version: 0.1
#### Date published: TBD

## Purpose

Protecting SLP token users against scams and harmful token content can be achieved by delegating review and certification of token legitamacy to 3rd parties.  Using the protocol specified herein any individual or entity can publish a simple vote for a token's legitamacy to the Bitcoin Cash blockchain.  Wallets and block explorer builders can then filter the voting data down to individuals whom they trust and then compute their own arbitrary trust score used to protect their users.  

## Voting Procedure

Votes can be cast only in favor of a token's legitamacy which can be revoked at any time.  Votes and revocations shall be sent with a small dust or tip amount to the token's genesis address so that they can be alerted of the change in certification status.  Each vote/revokation certificate should be specific to a token.json document, this way any updates to token.json will require a new vote certificate.  The vote for a previous token.json certificate is not required to be revoked.

A vote or revocation exists in the form of a small OP_RETURN certificate containing pointer towards a `tokenId` and a `file_sha256_bytes` for a specific `token.json` document version. 

#### Vote Transaction Format
| Index | Inputs                        | Output scriptPubKey ASM                                     | Output amount |
| ----- | ----------------------------- | ----------------------------------------------------------- | ------------- |
| 0     | voter's input w/ sig & pubkey | `OP_RETURN` `TTP\x00` `1` `tokenId_32bytes` `BfpId_32bytes` | 0 sat         |
| 1     | any input                     | Genesis output receiver                                     | 546 sat min   |
| ...   | ...                           | ...                                                         | ...           |
| n     | any input                     | change or any other output                                  | any           |

#### Revocation Transaction Format
| Index | Inputs                        | Output scriptPubKey ASM                                     | Output amount |
| ----- | ----------------------------- | ----------------------------------------------------------- | ------------- |
| 0     | voter's input w/ sig & pubkey | `OP_RETURN` `TTP\x00` `0` `tokenId_32bytes` `BfpId_32bytes` | 0 sat         |
| 1     | any input                     | Genesis output receiver                                     | 546 sat min   |
| ...   | ...                           | ...                                                         | ...           |
| n     | any input                     | change or any other output                                  | any           |

NOTES: 
1. Voter's input signature must use sighash flag `ALL`, `SINGLE`, `ALL|ANYONECANPAY`, or `SINGLE|ANYONECANPAY`.
2. Any input script may be used however the first two elements of scriptSig must be a signature followed by the voter's pubkey.
3. Voters wanting to use a multisig address can use key aggregation with Schnorr

