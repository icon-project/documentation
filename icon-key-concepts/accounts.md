---
title: "Accounts"
excerpt: ""
---

## Account
Accounts represent identities of participants in the blockchain network, that is, users and contracts. Accounts are used to identify the owner of transactions and specify the destinations of transactions. All accounts can have an ICX balance.

There are two types of accounts in ICON, accounts that are associated with users, often dubbed as Externally Owned Accounts (EOA), and Smart Contract (SCORE) Accounts. An EOA address starts with "hx" followed by a 20-byte hex string, while a Smart Contract Account address starts with "cx" followed by a 20-byte hex string. 

EOA has an associated private key, can sign and send a transaction, therefore can be the owner of transactions. However, a contract address does not have a private key, and cannot initiate a transaction. A contract address can only be the destination of transactions.  

There are some special, pre-defined accounts, such as:
 - Governance contract
 - Public Treasury

In order to deploy a SCORE on the ICON network or make a transaction, one must hold a valid EOA. Furthermore, when we say account in this document, it means EOA. The terms of EOA, wallet, and keystore are not exactly the same, but are often used interchangeably. 

An account is cryptographically defined by a private-public key pair, and the account address can be derived from its public key by taking the last 20-bytes of the SHA3-256 hash of the public key. Creating an account is equivalent to creating a key pair, and the account can be exported as a file known as a keystore file. 

## Keystore file
A keystore file is a JSON text file containing your private key and address. The private key is encrypted with the password that you enter when you create an account or keystore file. If you lose your keystore file and password, there is no way to recover them. You will lose your account and the assets you own.

A keystore file is used to authenticate a user. Every transaction must be signed with your private key. If your private key is leaked, anyone possessing the private key can access your account and sign the transaction on your behalf. Therefore, it is always recommended to use a keystore file instead of a plain private key. 


