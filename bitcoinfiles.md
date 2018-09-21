![Simple Ledger Protocol](images/SLP-logo-solid-200.png)



# Bitcoin Files Protocol Specification

### Specification version: 0.1
### Date published: September 21, 2018
### URL: [bitcoinfiles.com](bitcoinfiles.com)

## Authors
James Cramer

## Acknowledgements
Mark Lundeberg, for further simplifying the protocol

Jonald Fyookball, for various comments and considerations 

Calin Culianu, for EC SLP software implementation support

## Introduction
The following presents a simple protocol for uploading and downloading binary files to the blockchain.  The motivation for this protocol was driven by lack of reliable and anonymous file upload web services with a good free API, so BFP was born.  The original purpose of this protocol is to facilitate the simple uploading of JSON documents which can be related to a specific SLP token in the token's GENESIS transaction.

BFP is a separate protocol from SLP Tokens, but its design uses a DAG to link file chunks and metadata together over multiple transactions so in that way it is similar to SLP tokens.  

## Protocol Overview

This protocol describes the ruleset for uploading and downloading of files of any size that are stored on the Bitcoin Cash blockchain.  

### File Uploads

Files are uploaded to the blockchain in a series of chunks encapsulated within OP_RETURN messages located at vout=0 (i.e. the first output) in each file chunk's respective transaction.  The file chunks reference each other using the vout=1 output as a pointer to the location of the next data chunk.  The file can be shared with anyone by simply sharing the transaction id of the file's last uploaded chunk, which also contains optional file metadata.

The following illustration shows an example multi-part file upload using this protocol:

![bfp-fig-1](images/bfp-fig-1.png)

The transaction id belonging to the last transaction made in the series of uploaded file data chunks is the file's handle.  This final transaction contains a special set of optional file metadata parameters, most importantly the number of chunks associated with the file is needed so an implementation knows how many data chunks to download.  Other properties for the file can be included as shown in the Script specification below.

It is recommended that an initial funding transaction be created (as shown in the above figure) to cover the costs of all subsequent transactions.  The reason for this is that all of the transactions can be signed one after another, then all of the signed transactions can be submitted to the network as a batch.  This also reduces the overall cost for the file upload since each file chunk's transaction will only have 1 input.

#### V1 Protocol Rules

1. **Chunk Order:** Chunks shall be ordered in the order that the transactions are signed.  The first chunk of the file represents the first part of the file and is signed first.  The last chunk of the file is signed last and may be placed within the final metadata transaction if there is sufficient room.

2. **Metadata Transaction OP_RETURN:** The last and final signed transaction for the file must contain an OP_RETURN message containing metadata located at output index 0 (i.e., vout:0).  The required format is:

   * `OP_RETURN <lokad_id_int> <bfp_version_int = 0x01> <chunk_count_int> <filename_no_extension_utf8*> <file_extension_utf8*> <size_int*> <file_hash_int*> <chunk_X_data_bytes*>`
   * If the file includes only 1 data chunk *AND* that data chunk is included within the metadata transaction then no further requirements for locating file data exist.
   * If the file includes 1 or more data chunks *AND* the chunk data is not included in this metadata transaction then first input to this transaction (i.e., vin: 0) shall be used to locate the first data chunk.

3. **Chunk Transaction OP_RETURN:** For any pure data chunk transaction output index 0 (i.e., vout: 0) shall contain the OP_RETURN message with the following format:

   * `OP_RETURN <chunk_X_data_bytes>`

4. **Chunk Transaction Baton:** For any pure data chunk transaction a baton is used so that a reference pointer can be created to the file's next chunk or metadata. Output index 1 (i.e., vout: 1) shall contain the UTXO dust that shall be spent in the next transaction as the first input (i.e., vin: 0).  The next transaction after a pure data chunk transaction can be either another pure data chunk transaction OR at the file's final metadata transaction.

5. **Endianness:** All data pushes, other than the Lokad identifier, shall be pushed using big-endian data byte ordering.


**Notes on OP_RETURN syntax presented above:** 

1. Data push opcodes are not presented above. For each variable the appropriate data push code shall be utilized.  Opcodes 

2. Data fields are represented using the variable's name in angle brackets (i.e.,  `<some_name>` )

3. The data encoding for each variable is included as the final part of the variable name.

4. Optional fields are indicated using `*` at the end of the field's name, and can be left empty using the push code `0x4c 0x00`. 


### File Download Procedure

Files are located on the blockchain using the transaction id of the file's metadata OP_RETURN message. In order to download the full file content the software implementation simply needs to follow the following steps: 

1. Download the file's metadata transaction
2. Parse for Metadata OP_RETURN message located at the first output (i.e., vout:0) within that transaction
3. If there is only 1 chunk and a chunk is provided within the metadata message then the procedure is complete.
4. If there are more chunks that need to be downloaded then use the transaction id specified at vin:0 to find the next data chunk and repeat this process until all of the chunks have been downloaded.  Validation should be performed at each step making sure the protocol rules were followed.
5. Parse pure data chunks using the "Chunk Transaction OP_RETURN" format specified above.
6. Reconstruct the file's data chunks using the first signed = first chunk rule.



### Other Considerations

1. Set a limit for file upload sizes to encourage wise use of blockchain space
2. Network rules currently limit the number of chained transactions to 25 per block, this limits the data throughput of this protocol to slightly more than 5kB files.