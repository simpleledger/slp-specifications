# SLP Chat D-App Specification - DRAFT

***A decentralized peer-to-peer group chat system using SLP tokens***

**Last updated: May 19, 2020**



## Introduction

SLP Chat is a decentralized application (d-app) for participating in peer-to-peer group chat rooms with end-to-end encryption.  The application is open source and is designed in a way to permit interoperation with other implementations without requiring dependence on third-party services.  Chat room users can restore their chat group history by simply restoring their 12-word mnemonic seed phrase as long as one other group member is online to share chat history.

The unique and highly motivating features this application will provide include:

* group chat rooms with end-to-end encryption
* native tipping with SLP tokens
* integrated workflows for interacting with smart contracts with chat group participants

Group chat rooms are easy to setup and only require issuance of an [NFT1 Group](https://slp.dev/specs/slp-nft-1/) token by a chat room administrator, and users are added to the chat group be creating of NFTs from the NFT1 group which are sent to the chat room users.  The public key for each chat room member is embedded in the NFT genesis transaction and is used as the peer's identity in the peer-to-peer network and end-to-end message encryption.

<img src="./figures/figure_1.png" alt="figure_1" style="zoom:50%;" />



Once users are holding one or more chat room NFT tokens in the SLP chat app's integrated wallet users can connect and securely message with the other online members of their groups.  The following figure illustrates the relevant p2p network topology of users belonging to different chat groups.

<img src="./figures/figure_2.png" alt="figure_2" style="zoom:50%;" />



## Application Architecture

A reference client *(currently in alpha)* for the SLP chat application has been built using TypeScript and ReactJS UI framework.  The core of the chat application leverages [IPFS](https://ipfs.io) for peer-to-peer messaging and files storage, and uses [SLPDB](https://github.com/simpleledger/SLPDB) for updating SLP token information and wallet balance information.

![figure_3](./figures/figure_3.png)

Peer-to-peer connections between chat room users in the browser is accomplished using WebRTC and is handled automatically by the [libp2p](https://github.com/libp2p/js-libp2p) javascript library.  The chat room administrator must select a rendezvous point for the chat room peers to initiate and establish the p2p connection between users using one or more [WebRTC-star](https://github.com/libp2p/js-libp2p-webrtc-star) servers.   To be clear, no chat messages are not routed through this webRTC-star server, its purpose is purely for the purpose of peer discovery and introductions.

Both SLPDB and the WebRTC services can be configured by the chat room administrator and are not dependent on a third-party.  Most group admins would likely just leverage public WebRTC-star and SLPDB servers.



## SLP Token Data Scheme

SLP Chat stores a small amount of information about chat groups and its chat group members in the "document url" and "document hash" fields of SLP Genesis transactions.  The following diagram illustrates the types of data stored within the NFT1 Group and NFT child token Genesis transactions.

New chat groups can be publicly listed as a known chat group ID by including the SLP chat application ID (currently not yet set in stone) within the "document hash" field of the chat group NFT1 Group.  Not including the SLP chat app ID will result in a chat group that will not be listed as a known chat group and would require manually adding the group ID in the SLP application client.

<img src="./figures/figure_4.png" alt="figure_4" style="zoom:50%;" />



## IPFS Data Scheme

Users communicate with peers in real-time using libp2p pubsub.  There are currently only three types of messages sent via pubsub:

* `GROUP_SEND` - Used for sending a new group messages to connected peers
* `NAME_REQUEST` - Used for requesting the current "myIpns" directory CID for a particular peerId
* `NAME_RESPONSE` - Used for responding with the best known "myIpns" directory CID for a particular peerId

Additionally, users store and pin their own messages on a locally running IPFS node.  In the reference SLP chat application there is an embedded IPFS node running within the javascript web application, and all of the data is stored within an IndexedDB and local storage.  The directory structure for the files stored by each user is illustrated below.  

The IPFS files repository contains two main directories, `myIpns` and `files`.  

* `myIpns` is a lightweight directory used as an index for finding all the directories storing a user's group chat messages or admin group settings information.  The directory is discovered via a custom name resolution system kind of like IPNS.

* `files` is where a user's actual chat files are stored.  A user only stores their own chat messages within the files directory structure.  Messages from other users within the group are stored but are only found by CID and not through the files directory structure.  Each message is stored within the respective group chat id folder having a file name of the message timestamp.

<img src="./figures/figure_5.png" alt="figure_5" style="zoom:50%;" />



### Messaging and File Formats

The format for all pubsub messages between peers and IPFS files are defined using the following protobuf message definitions.  Some of the message definitions have not yet been implemented in the reference SLP chat application and are subject to change by the time this initial version of this specification is published.

```protobuf
  syntax = "proto3";

  // This message is used by all users located in the users's MFS
  // at: /myIpns/groups/<group id hex>/index
  message ChatGroupIndexFile {
    string user_msg_dir_cid = 1;
    string admin_group_info_cid = 2;  // used by admins only
    string handle = 3;
  }

  // This message can be optionally included within the Group Chat NFT1 Genesis transaction
  // documentURL field as a binary payload.  It's primary purpose is to provide a pointer
  // to the IPNS peerId (secp256k1 pubkey) that is storing the mutable information and rules 
  // about the group chat that the chat users can resolve.
  //
  // Note that this message is separate from the 32 byte SLP chat application ID which
  // should be included within the document hash field of the same NFT1 Group Genesis
  // transaction.
  //
  // Currently this is not implemented anywhere.
  //
  message ChatGroupGenesisUrl {
    bytes admin_pubkey = 1;
  }

  // Group admin only: this is an optional document to provide
  // general information, rules etc., about the particular group chat.
  //
  // Currently this is not implemented anywhere
  //
  message ChatGroupInfoFile {
    string group_name = 1;
    string group_description = 2;
    string url = 3;
    bool require_encryption = 4;
    bool confidential_conversation = 5;
    repeated string swarm_addresses = 6;
    repeated string slpdb_addresses = 7;
  }

  // This message is for each chat user and shall be included
  // within the documentURL field of the user's NFT Genesis transaction.
  message ChatUserGenesisUrl {
    bytes user_pubkey = 1;
  }

  // This message represents the actual chat messages themselves, it is used
  // for both the payload within a chat message delivered via pubsub or when 
  // fetch as a file from IPFS when resolving chat history
  message ChatMessagePayloadPb {
    enum ChatFileType {
      UTF8 = 0;
      FILE = 1;
      ECIES_SECP256K1_UTF8 = 2;
      ECIES_SECP256K1_FILE = 3;
    }
    message ChatMessagePayloadItemPb {
        bytes payload = 1;
        bytes pubkey = 2;
    }
    ChatFileType type = 1;
    repeated ChatMessagePayloadItemPb payload = 2;
    string filename = 3;
  }

  // This is an envelope indicating which
  // type of pubsub message is being sent
  message PubsubEnvelope {
    enum PubsubType {
      GROUP_SEND = 0;
      NAME_REQUEST = 1;
      NAME_RESPONSE = 2;
    }
    PubsubType type = 1;
    bytes payload = 2;
  }

  // This is the pubsub payload for GROUP_SEND
  message PubsubGroupSend {
    bytes groupid = 1;
    int64 time = 2;
    string msg_cid = 3;
    string msg_dir_cid = 4;
    string name_dir_cid = 5;
    bytes name_dir_cid_sig = 6;
    bytes message = 7;
  }

  // This is the pubsub payload for NAME_REQUEST
  message PubsubNameRequest {
    string peerid = 1;
    int64 time = 2;
    string prev_cid = 3;
    bytes prev_sig = 4;
  }

  // This is the pubsub payload for NAME_RESPONSE
  message PubsubNameResponse {
    string peerid = 1;
    string cid = 2;
    int64 time = 3;
    bytes cid_sig = 4;
  }

  // This data structure can be used to persist the Mapping of
  // peer chat content to be pinned
  message ChatRoomPinMap {
    message MapItem {
      string peerid = 1;
      bytes groupid = 2;
      string cid = 3;
    }
    repeated MapItem pinset = 1;
  }

  // This data structure can be used to persist the Mapping of
  // PeerId->myIpns dirctory of each peer
  message NamePinMap {
    message MapItem {
      string peerid = 1;
      string cid = 2;
      int64 time = 3;
      bytes cid_sig = 4;
    }
    repeated MapItem pinset = 1;
  }
```

