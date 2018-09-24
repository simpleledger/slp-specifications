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

MatterBCH, for suggesting IFPS metadata in file uploads

____, for suggesting external file link type message

## 1. Introduction
The following presents a simple protocol for adding files to the Bitcoin Cash blockchain.  The protocol allows for creating immutable hyperlinks pointing to on-chain and off-chain data storage providers.  The motivation for this protocol was driven by lack of reliable and anonymous file upload web services that have a good API for creating a public shareable file URL.  

The original purpose of such a protocol is to facilitate the simple uploading of JSON documents for an SLP token's GENESIS transaction. BFP is a separate protocol from SLP Tokens, but its design uses a DAG to link file chunks and metadata together over multiple transactions so in that way it is similar to SLP tokens.

## 2. Protocol Requirements

This protocol specification describes the requirements for handling files linked to the Bitcoin Cash blockchain.  Considerations have been given to both on-chain and off-chain file storage facilities.

### 2.1 BFP Message Types

Unique BFP message types are used to represent different constructs in the BFP filesystem.  In any BFP OP_RETURN message the message type is represented by the field `<bfp_msg_type>`.  There are currently three types of BFP message types, including:

- On-chain file storage device (flagged by `bfp_msg_type = 0x01`)
- Off-chain file storage device (flagged by `bfp_msg_type = 0x02`)

### 2.2 On-chain File Storage Device (Message Type = 0x01)

Files are uploaded to the blockchain in a series of chunks encapsulated within OP_RETURN messages located at vout=0 (i.e. the first output) in each file chunk's respective transaction.  The file chunks reference each other using the vout=1 output as a pointer to the location of the next data chunk.  The file can be shared with anyone by simply sharing the transaction id of the file's last uploaded chunk, which also contains optional file metadata.

The following illustration shows an example multi-part file upload using this protocol:

![bfp-fig-1](images/bfp-fig-1.png)

The transaction id belonging to the last transaction made in the series of uploaded file data chunks is the file's handle.  This final transaction contains a special set of optional file metadata parameters, most importantly the number of chunks associated with the file is needed so an implementation knows how many data chunks to download.  Other properties for the file can be included as shown in the Script specification below.

It is recommended that an initial funding transaction be created (as shown in the above figure) to cover the costs of all subsequent transactions.  The reason for this is that all of the transactions can be signed one after another, then all of the signed transactions can be submitted to the network as a batch.  This also reduces the overall cost for the file upload since each file chunk's transaction will only have 1 input.

#### 2.2.1 Rules for on-chain file uploads

1. **Chunk Order:** Chunks shall be ordered in the order that the transactions are signed.  The first chunk of the file represents the first part of the file and is signed first.  The last chunk of the file is signed last and may be placed within the final metadata transaction if there is sufficient room.

2. **Metadata Transaction OP_RETURN:** The last and final signed transaction for the file must contain an OP_RETURN message containing metadata located at output index 0 (i.e., vout:0).  The required format is:
   * Since Block #550,000: `OP_RETURN <lokad_id_int = 'BFP\x00'> <bfp_msg_type = 0x01> <chunk_count_int> <filename_no_extension_utf8*> <file_extension_utf8*> <size_bytes_int*> <file_sha256_int*> <ipfs_file_hash_int*> <chunk_X_data_bytes*>`
   * If the file includes only 1 data chunk *AND* that data chunk is included within the metadata transaction then no further requirements for locating file data exist.
   * If the file includes 1 or more data chunks *AND* the chunk data is not included in this metadata transaction then first input to this transaction (i.e., vin: 0) shall be used to locate the first data chunk.

3. **Chunk Transaction OP_RETURN:** For any pure data chunk transaction output index 0 (i.e., vout: 0) shall contain the OP_RETURN message with the following format:
   * Since Block #550,000: `OP_RETURN <chunk_X_data_bytes>`

4. **Chunk Transaction Baton:** For any pure data chunk transaction a baton is used so that a reference pointer can be created to the file's next chunk or metadata. Output index 1 (i.e., vout: 1) shall contain the UTXO dust that shall be spent in the next transaction as the first input (i.e., vin: 0).  The next transaction after a pure data chunk transaction can be either another pure data chunk transaction OR at the file's final metadata transaction.

5. **Endianness:** All data pushes, other than the Lokad identifier, shall be pushed using big-endian data byte ordering.

**Notes on OP_RETURN syntax presented above:** 

1. Data push opcodes are not presented above. For each field variable an appropriate data push code shall be utilized.  Opcodes rules for data pushes 

2. Data fields are represented using the variable's name in angle brackets (i.e.,  `<some_name>` )

3. The data encoding for each variable is included as the final part of the variable name.

4. Optional fields are indicated using `*` at the end of the field's name, and can be left empty using the push code `0x4c 0x00`. 

#### 2.2.2 Procedure for downloading on-chain files

Files are located on the blockchain using the transaction id of the file's metadata OP_RETURN message. In order to download the full file content the software implementation simply needs to follow the following steps: 

1. Download the file's metadata transaction
2. Parse for Metadata OP_RETURN message located at the first output (i.e., vout:0) within that transaction
3. If there is only 1 chunk and a chunk is provided within the metadata message then the procedure is complete.
4. If there are more chunks that need to be downloaded then use the transaction id specified at vin:0 to find the next data chunk and repeat this process until all of the chunks have been downloaded.  Validation should be performed at each step making sure the protocol rules were followed.
5. Parse pure data chunks using the "Chunk Transaction OP_RETURN" format specified above.
6. Reconstruct the file's data chunks using the first signed = first chunk rule.

### 2.3 Off-chain File Storage Device (Message Type = 0x02)

#### 2.3.1 Rules for off-chain uploads

SOMEONE WHO KNOWS IPFS ETC TO FILL THIS IN

#### 2.3.2 Procedure for downloading off-chain files

SOMEONE WHO KNOWS IPFS ETC TO FILL THIS IN

## 3. File Handler Prefixes

This part of the specification in not a protocol rule and is only a recommendation for an improved user experience.  It is recommended that a prefix of "bitcoinfile:" be used for transaction ids representing either on-chain or off-chain file.  In the future the concept of folders can be added to this protocol and the prefix of "bitcoinfiles:" should be used.  

For example:

`bitcoinfile:<txid-of-a-file>`

`bitcoinfile:<txid-of-a-folder>`

The usage of a transaction id prefix shall have no impact on the protocol rules, and implementations should completely ignore the prefix if it is provided by the user.  Only the transaction id matters when determining the content of a BFP message.

## 4. Other Considerations

1. Network rules currently limit the number of chained transactions to 25 per block, this limits the data throughput of this protocol to slightly more than 5kB files.  Implementations will need consider the number of chained transactions a UTXO may already has before creating a file upload.  For example, the file upload may be limited to fewer than 25 transactions if the user has made several transactions prior to the file uploads within the same block height. 

2. Set a limit for file upload sizes to encourage wise use of blockchain space

3. In the future the concept of file folders could be introduced.  There are at least 2 approaches that could be taken for folders.

   a) List multiple file txids in OP_RETURN messages.

   b) Spend UTXOs sitting at file transactions in a new transaction with a new "folder" BFP message type.  

## 5. Specification Updates

**September 24, 2018**

* Protocol versioning to be dependent on block height.  First version starts at block #550,000.
* Added BFP Message Type ( `bfp_msg_type` ) and delineated specification for each message type
* Added off-chain storage feature BFP Message Type = 0x02