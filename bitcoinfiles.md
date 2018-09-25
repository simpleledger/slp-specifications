![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# Bitcoin Files Protocol Specification

### Specification version: 0.2
### Date published: September 21, 2018

### URL: [bitcoinfiles.com](bitcoinfiles.com)

## Authors
James Cramer

## Acknowledgements
Mark Lundeberg, for further simplifying the protocol

Jonald Fyookball, for various comments and considerations

Calin Culianu, for EC SLP Wallet implementation assistance

Attila Aros, for suggesting IFPS metadata in file uploads

hapticpilot, for IPFS and URI formatting suggestions

## 1. Introduction
The following presents a simple protocol for adding files to the Bitcoin Cash blockchain.  The protocol allows for creating immutable hyperlinks pointing to on-chain and off-chain data storage providers.  The motivation for this protocol was driven by lack of reliable and anonymous file upload web services that have a good API for creating a public shareable file URL.

The original purpose of such a protocol is to facilitate the simple uploading of JSON documents for an SLP token's GENESIS transaction. BFP is a separate protocol from SLP Tokens, but its design uses a DAG to link file chunks and metadata together over multiple transactions so in that way it is similar to SLP tokens.

## 2. Protocol Requirements

This protocol specification describes the requirements for handling files linked to the Bitcoin Cash blockchain.  Considerations have been given to both on-chain and off-chain file storage facilities.

### 2.1 BFP Types

Unique BFP message types are used to represent different constructs in the BFP filesystem.  In any BFP OP_RETURN message the message type is represented by the message field called `bfp_msg_type`.  There are currently three types of BFP constructs, including:

- An on-chain file (flagged by `bfp_msg_type = 0x01`)
- An off-chain file hyperlink (flagged by `bfp_msg_type = 0x02`)
- A folder (flagged by `bfp_msg_type = 0x03`)

### 2.2 On-chain File (Message Type = 0x01)

Files are uploaded to the blockchain in a series of chunks encapsulated within OP_RETURN messages located at vout=0 (i.e. the first output) in each file chunk's respective transaction.  The file chunks reference each other using the vout=1 output as a pointer to the location of the next data chunk.  The file can be shared with anyone by simply sharing the transaction id of the file's last uploaded chunk, which also contains optional file metadata.

The following illustration shows an example multi-part file upload using this protocol:

![bfp-fig-1](images/bfp-fig-1.png)

The transaction id belonging to the last transaction made in the series of uploaded file data chunks is the file's handle.  This final transaction contains a special set of optional file metadata parameters, most importantly the number of chunks associated with the file is needed so an implementation knows how many data chunks to download.  Other properties for the file can be included as shown in the Script specification below.

It is recommended that an initial funding transaction be created (as shown in the above figure) to cover the costs of all subsequent transactions.  The reason for this is that all of the transactions can be signed one after another, then all of the signed transactions can be submitted to the network as a batch.  This also reduces the overall cost for the file upload since each file chunk's transaction will only have 1 input.

#### 2.2.1 Rules for on-chain file uploads

1. **Chunk Order:** Chunks shall be ordered in the order that the transactions are signed.  The first chunk of the file represents the first part of the file and is signed first.  The last chunk of the file is signed last and may be placed within the final metadata transaction if there is sufficient room.

2. **Metadata Transaction OP_RETURN:** The last and final signed transaction for the file must contain an OP_RETURN message containing metadata located at output index 0 (i.e., vout:0).  The required format is:
   *  `OP_RETURN <lokad_id_int = 'BFP\x00'> <bfp_msg_type = 0x01> <chunk_count_int> <filename_no_extension_utf8*> <file_extension_utf8*> <size_bytes_int*> <file_sha256_int*> <file_uri_utf8*> <chunk_X_data_bytes*>`
   * If the file includes only 1 data chunk *AND* that data chunk is included within the metadata transaction then no further requirements for locating file data exist.
   * If the file includes 1 or more data chunks *AND* the chunk data is not included in this metadata transaction then first input to this transaction (i.e., vin: 0) shall be used to locate the first data chunk.

3. **Chunk Transaction OP_RETURN:** For any pure data chunk transaction output index 0 (i.e., vout: 0) shall contain the OP_RETURN message with the following format:
   *  `OP_RETURN <chunk_X_data_bytes>`

4. **Chunk Transaction Baton:** For any pure data chunk transaction a baton is used so that a reference pointer can be created to the file's next chunk or metadata. Output index 1 (i.e., vout: 1) shall contain the UTXO dust that shall be spent in the next transaction as the first input (i.e., vin: 0).  The next transaction after a pure data chunk transaction can be either another pure data chunk transaction OR at the file's final metadata transaction.

5. **Endianness:** All data pushes, other than the Lokad identifier, shall be pushed using big-endian data byte ordering.

#### 2.2.2 Procedure for downloading on-chain files

Files are located on the blockchain using the transaction id of the file's metadata OP_RETURN message. In order to download the full file content the software implementation simply needs to follow the following steps:

1. Download the file's metadata transaction
2. Parse for Metadata OP_RETURN message located at the first output (i.e., vout:0) within that transaction
3. If there is only 1 chunk and a chunk is provided within the metadata message then the procedure is complete.
4. If there are more chunks that need to be downloaded then use the transaction id specified at vin:0 to find the next data chunk and repeat this process until all of the chunks have been downloaded.  Validation should be performed at each step making sure the protocol rules were followed.
5. Parse pure data chunks using the "Chunk Transaction OP_RETURN" format specified above.
6. Reconstruct the file's data chunks using the first signed = first chunk rule.

### 2.3 Off-chain File (Message Type = 0x02)

#### 2.3.1 Rules for creating hyperlinks to off-chain files

Immutable hyperlinks to files stored off-chain can be created using Type 0x02 BFP message.  The file's URI shall be used to identify the external storage device and the relative location of the file on that device.

1. **Metadata Transaction OP_RETURN:** A single transaction containing an OP_RETURN message at output index 0 (i.e., vout:0) shall be provided.  The required format is:
   -  `OP_RETURN <lokad_id_int = 'BFP\x00'> <bfp_msg_type = 0x02> <file_uri_utf8> <file_uri_metadata_utf8> <filename_no_extension_utf8*> <file_extension_utf8*> <size_bytes_int*> <file_sha256_int*> `

#### 2.3.2 Procedure for downloading off-chain files from hyperlinks

Downloading files should be handled per the requirements for each specific off chain storage system.  The requirements should be deterministic based on the provided URI.

#### 2.3.2.1 IPFS downloads

[IPFS](https://ipfs.io) is a system for storing and distributing files. The key benefit of IPFS is decentralization and consistent addressability.

An IPFS hash can be included to make it easier for users to download content more efficiently, instead of querying for chunk transactions on the blockchain. Since the IPFS hash is uniquely determined by the file content (and a few other settings explained below shortly), then this is a perfect and accurate replacement for the download operation.

- User agent *may* choose to query an IPFS gateway of their choice after retrieving the metadata transaction.

There are numerous public gateways available that would be suitable options for resolving IPFS links.

For example, if a metadata transaction contained the IPFS hash `Qmc5gCcjYypU7y28oCALwfSvxCBskLuPKWpK4qpterKC7z`, then the user agent could download this file from an IPFS gateway via a URL of the form: `https://gateway.ipfs.io/ipfs/Qmc5gCcjYypU7y28oCALwfSvxCBskLuPKWpK4qpterKC7z`

- User browser agent *may* choose to query an IPFS swarm directly instead using a pure Javascript/Browser implementation such as [js-ipfs](https://github.com/ipfs/js-ipfs).

- User agent *may* to set a time limit for how long they are willing to wait for IPFS to resolve the content, and instead opt for retrieving the individual chunk transactions to serve their request instead.

If the content is not retrievable or not found on IPFS, then the user *may* re-upload the content to IPFS after retrieving the content bytes from the transaction chunks. The next section highlights the considerations for performing an upload as there are some IPFS specific flags that need to be set consistently to arrive at the same IPFS hash.

#### 2.3.2.2 IPFS uploads

IPFS hashes are uniquely determined by a few parameters, not simply the sha256 hash of the file content. This is unavoidable since IPFS provides different storage options, identifier versioning, and also needs some flexibility for the future.

IPFS options (which determine the IPFS hash created with `ipfs add -n`)
* Chunking algorithm (--chunker option)
* DAG format (--trickle option)
* CID version (--cid-version option)
* Hashing algorithm (--hash option)
[Source](https://discuss.ipfs.io/t/how-to-calculate-file-directory-hash/777/4)

**Defaults**

--chunker

Default: `size-262144` (1024*256)

--trickle

Default: false

--cid-version

Default: CIDv0 (Starts with 'Qm...')

Note: this is a [self-describing format](https://github.com/ipld/cid#versions) and therefore can be uniquely determined by looking at the hash prefix

--hash

Default: sha256

At the time of writing, the latest version of IPFS is `ipfs version 0.4.17` and the defaults described above are accurate for this version.

It is *recommended* that user agents calculate the IPFS hash for a file using `ipfs add -n` (and the necessary parameters). This will ensure maximum compatibility for other users, in the event that the content has been garbaged collected from all live IPFS nodes and a user wishes to make the content available again under the same IPFS hash.

The facilite being able to easily recreate the IPFS hash, we are *recommending* that users include the optional `<file_uri_metadata_utf8>` to inform clients what IPFS hashing options were used.

The parameter `<file_uri_metadata_utf8>` is a tab delimited utf8 string of the form:

`<ipfs_options_info_version|<known_working_ipfs_str_version>|<chunker_str_utf8>|<trickle_bool_int>|<hash>|`

Note: The --cid-version is not included, since it is a self-describing format and can be identified from the IPFS hash itself.

`ipfs_options_info_version` is a microformat, here set to `1` to be used later in the event we want to change the contents of this tab delimited string.

Examples:

` `: Any empty string is considered to use the default (best effort guesses of what settings were used)
`||||`: This means it is not provided (use defaults, best effort guesses).
`1||||`: This means it is not provided (use defaults, best effort guesses).
`1|0.4.17||||`: Version is set, but agent did not provide any additional information
`1|0.4.17|size-262144||`: Version is set, and chunker option set
`1|0.4.17|size-262144|1|`: Version is set, chunker option set, and trickle is set to true
`1||size-262144|0|sha256`: Version unknown, chunker option set, trickle is set to false, and sha256 is the hashing algorithm.

It is *recommended** that the user agent sets all the command line options for `ipfs add -n` and then provides these values here explicitly to enable others to create the precise hash.  On the other hand, the defaults are not expected to change anytime soon, therefore as long as the user is using an unmodified version of ipfs, then they should be able to easily obtain the same hash.


### 2.4 Folders (Message Type = 0x03)

A folder message type stores one or more transaction hash/ids belonging to other files and folder message types.  This type of message simply provides a list of transaction hashes

#### 2.4.1 Rules for creating folders

1. **Metadata Transaction OP_RETURN:** The last and final signed transaction for the file must contain an OP_RETURN message containing metadata located at output index 0 (i.e., vout:0).  The required format is:
   * `OP_RETURN <lokad_id_int = 'BFP\x00'> <bfp_msg_type = 0x03> <list_page_count> <folder_name*>  <folder_description*> <txid_0_int> ... <txid_i_int*> ... <txn_id_n>`

2. **Transactions List Page OP_RETURN:** Since a list of transaction hashes may be as long as required additional OP_RETURN space may need to be provided.  The number of list pages should be specified to be greater than 1 within the metadata OP_RETURN in order to indicate
3. **List Page Transaction Baton:**  For any list page transaction a baton is used so that a reference pointer can be created to the file's list page or folder metadata transaction. Output index 1 (i.e., vout: 1) shall contain the UTXO dust that shall be spent in the next transaction as the first input (i.e., vin: 0).  The next transaction after a list page transaction can be either another list page transaction OR at the folder's final metadata transaction.

#### 2.4.2 Procedures for discovering files within a folder

The rules for determining the files in a folder are simple.  An implementation shall parse the Metadata OP_RETURN message and any upstream List Page OP_RETURN messages for the transaction hash/Ids which should point to valid BFP files or folders.

## 3. OP_RETURN Syntax and Format Requirements

1. Data fields are represented using the field's name within angle brackets (i.e.,  `<some_field_name>` )
2. Data push opcodes are not presented above. For each field an appropriate data push code shall be utilized.  Opcodes rules for data pushes
3. The data encoding for each variable is included as the final part of the variable name.
4. Optional fields are indicated using `*` at the end of the field's name, and can be left empty using the push code `0x4c 0x00`, or `0x4d 0x00 0x00`, or `0x4e 0x00 0x00 0x00 0x00`.
5. Bitcoin script allows a given byte array to be pushed in various ways, and we allow this in SLP as well. For example, it is valid to push a 4-byte chunk (like the Lokad ID) in four different ways: 0x04 [chunk], 0x4c 0x04 [chunk], 0x4d 0x04 0x00 [chunk], or 0x4e 0x04 0x00 0x00 0x00 [chunk].
6. Only opcodes 0x01 to 0x4e are permitted (after OP_RETURN). Note this means that not all push opcodes are allowed -- it is forbidden to use the empty-push opcode 0x00 (OP_0) or 1-byte literal push opcodes 0x4f-0x60 (OP_1 through OP_16 and OP_1NEGATE) anywhere in the OP_RETURN. For example, it is invalid to use 0x58 to push the number '8' in the 1-byte `chunk_count_int` field of the Type 0x01 file transaction message, even though in normal bitcoin script the opcode 0x58 is effectively equivalent to 0x01 0x08  (push [0x08]). For this reason some standard bitcoin script decompilers, that treat all push opcodes on equal footing, must not be used for parsing BFP transactions.

## 4. File URI Prefixes

This part of the specification in not a protocol rule and is only a recommendation for an improved user experience.  It is recommended that a prefix of "bitcoinfile:" be used for the transaction hash/id representing either on-chain or off-chain file.  In the future the concept of folders can be added to this protocol and the prefix of "bitcoinfiles:" should be used.

For example:

`bitcoinfile:<txid-of-a-file>`

`bitcoinfiles:<txid-of-a-folder>`

The usage of a transaction id prefix shall have no impact on the protocol rules, and implementations should completely ignore the prefix if it is provided by the user.  Only the transaction hash/id matters when determining the content of a BFP message.

## 5. Other Considerations

1. Network rules currently limit the number of chained transactions to 25 per block, this limits the data throughput of this protocol to slightly more than 5kB files.  Implementations will need consider the number of chained transactions a UTXO may already has before creating a file upload.  For example, the file upload may be limited to fewer than 25 transactions if the user has made several transactions prior to the file uploads within the same block height.

2. Set a limit for file upload sizes to encourage wise use of blockchain space

## 6. Specification Updates

**September 24, 2018**

* Added BFP Message Type ( `bfp_msg_type` ) and delineated specification for each message type
* Added off-chain storage BFP Message Type = 0x02
* Added folder BFP Message Type = 0x03
* Added file uri preferences