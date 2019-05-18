![Simple Ledger Protocol](images/SLP-logo-solid-200.png)



# Token Trust Protocol

#### Version: 0.1
#### Date published: TBD

## Purpose

Protecting SLP token users against scams and harmful token content can be achieved by delegating review and certification of token legitamacy to 3rd parties.  We expect that not all wallet and exchange maintainers will want to manually vet each token's legitimacy.  The protocol specified herein allows any individual or entity to publish a simple vote for a token's legitamacy to the Bitcoin Cash blockchain.  Wallets and block explorer builders can then filter the voting data down to individuals whom they trust and then compute their own arbitrary trust score used to protect their users.  

## Voting Procedure

Votes can be cast only in favor of a token's legitimacy which can be revoked at any time.  Votes and revocations shall be sent with a small dust or tip amount to the token's genesis address so the token owner can be alerted of the change in voting status.  Each vote/revocation certificate is linked to a specific to a token.json document version, this way any updates to the token.json file will require a new vote certificate.

A vote or vote revocation exists in the form of a small OP_RETURN certificate containing pointer towards a token ID (`tokenId`) and a bitcoin files protocol file ID (`BfpId_32bytes`) for a specific `token.json` (or `token.json.lzutf8`) document version.  Using bitcoin files ID instead of token document hash is more efficient 

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
1. Lokad Id for this voting protocol is `TTP\x00`.
2. Voter's input signature must use sighash flag `ALL`, `SINGLE`, `ALL|ANYONECANPAY`, or `SINGLE|ANYONECANPAY`.
3. Any input script may be used however the first two elements of scriptSig must be a signature followed by the voter's pubkey.
4. Voters wanting to use a multisig address can use key aggregation with Schnorr

## Token Document Updates
When token.json documents are updated, the votes linked to a previous token.json shall be considered immediately void so that token voters are not required revoke their previously created vote certificate.  Therefore, an issuer with a live token that is frequently used should be prepared to have his token re-certified by trusted ecosystem partners before posting his updated token document.  Vote certificates for a specific token document update may be issued on the blockchain before the document is actually updated shall be considered valid votes.
 
## Example Calculation Procedure

Here we demonstrate an arbitrary trust calculation determining whether our wallet software should display a warning symbol next to the token named "Super Saver Membership Token", or SSM. To get started, let's pretend that we trust any vote or revocations produced from all three of the individuals having the addresses:
    - Alice = bitcoincash:qabc
    - Bob = bitcoincash:qxyz
    - Chuck = bitxoincash:q123

### Trust Requirement 
This contrived example will require at least 2/3 votes from our trusted partners above.  Less than 2 votes will result in a warning symbol being display to our users and limited ability to display token metadata content.

### Step 1: Query BCH Blockchain for SSM token's Genesis Transaction
First we need to have the token's genesis transaction so that we can see where the token's Token Document (`token.json`) is stored.  The Token Document will contain important information like the purpose of the token, its icon, and have the contact information of the token's issuers.  This is simple to do because the token genesis transaction hash is the tokenId embedded within every token transfer.  We can simply use this transaction hash to download the token's Genesis transaction metadata.  Here is an example of how you would to this with SLPDB (TODO: example query).  

### Step 2: Use the Genesis Transaction data to find the latest Token Document
Now that we have our token's Genesis transaction we can look to see if this token includes a Token Document per the specification.  To do this we look at the Genesis field called `Token Document URL` which shall point to the storage location of the `token.json` resource. Since SLP Token Document specification requires use of bitcoin files protocol (BFP) the `Token Document URL` is required to be in the form `bitcoinfile:<32_byte_txid>`.  If the Token Document URL is not a BFP URI the we can immediately determine this is not a trusted token.  In this case we find that the token is using to a bitcoinfiles URL so we download this file.

Before we can consider this file to be valid we need to check to see if this document has been updated.  We look for an update to the `token.json` document by making a query for any BFP documents having a `prev_file_hash` field of the currently known BFP file.  The query looks like this (TODO: example query), which yields 2 results.   Per the Token Document specification we only consider the first document valid replacement.  The specification also states that only addresses listed within the "auth" field of `token.json` are permitted to perform the update.  Therefore, we are required consider the first occuring upload by an authorized user to be a valid update. We repeat this process of downloading and checking for valid updates until there we do not find any new updates.

In our case we found that the document was updated just one time, so now we are ready to look for any votes vouching for this latest token document.

### Step 3: Look for votes associated with the latest Token Document

We will use the latest document's 32-byte BFP hash in combination with the 32-byte token Id to query for any votes (or revoked votes) from our trusted partners.  Here is an example of how you would query for all votes in this example (TODO: example query).  

After running this query we find there are 2 votes, from our partners Alice and Bob, so we can confidently display this token's image to our wallet's users.