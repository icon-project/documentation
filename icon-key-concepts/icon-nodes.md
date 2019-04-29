---
title: "ICON Nodes"
excerpt: ""
---

This document presents what kinds of nodes are in the ICON network. 
Node means the computer server which participates in the blockchain protocol. All nodes keep the full or partial copy of blockchain data and execute transactions in the blocks for validation. So all nodes should be connected to the internet, and have their own address to identify themselves.

## Types of Nodes

There are three types of nodes in ICON network.
- **Peer nodes** can participate in the consensus protocol, so they can produce and propose a block.
- **Citizen nodes** just synchronize the blockchain data and relay the transaction to the Peer node.
- **Light clients** just synchronize the block headers for simple verification of transactions.

![types of nodes](types_of_nodes.png)

The user and application can access (query or send transactions) these nodes through ICONex wallet or ICON SDK. They can also access the nodes using JSON RPC directly as well.

## Peer

There are two types of Peer nodes, Public and Community Representative. 
As you see in the word "Representative", these nodes are supposed to be elected by ICONist or Community members. Details of the election are presented in [ICONSENSUS](https://icon.community/iconsensus/).

Peer nodes are the essential entities of ICON network. They have roles to produce and validate the blocks, which contains transactions transmitted into ICON network.
All transactions sent to ICON network are relayed or directed to Peer nodes. One of Peer nodes becomes a leader node, who has the right to propose a block on its turn. And the other Peer nodes validate the proposed block. The block is confirmed when 2/3 of Peer nodes agree on that block. Peer nodes will become a leader node in a pre-defined order, and produce one block on their turn.

### Public Representative (P-Rep)

Public Representatives are the consensus nodes that produce, verify blocks and participate in network policy decisions on the ICON Network. 

### Community Representative (C-Rep)

Community Representatives have a role of bridging ICON Network and their own Community blockchain.
They should relay the transactions and their proofs to the other blockchain. It is not decided whether C-Rep joins the consensus protocol or not. The detailed role of C-Rep will be defined after BTP (Blockchain Transfer Protocol) specification is finalized.

## Citizen

Citizen nodes synchronize the blockchain data from Peer nodes. Citizen is not just a simple data replication store but verifies every block data by executing the transactions in the block. Therefore, we can trust the sanity of the block data downloaded from the Peer nodes.
In addition, because Citizen does not participate in block generation, when it receives the transaction requests, Citizen relays the transaction requests to the Peer nodes.
With the above two main functions, Citizen is typically deployed as a service end-point of ICON Network. Citizen answers the queries from the users, and relays transactions to the Peer nodes. It is designed that no transactions and queries are supposed to be sent to the Peer nodes directly. This architecture keeps the Peer nodes focusing on consensus, that is, producing and validating blocks. In addition, limited access to Peer nodes makes the ICON network safer from attacks.
Since Citizen nodes verify the blocks and transactions, Exchanges or DApp operators are recommended to setup own Citizen nodes inside their network, rather than use publicly opened Citizen nodes outside their network.

## Light Client

TBA

