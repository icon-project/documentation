Account Management
==============

## Account
Accounts represent identities of participants in the blockchain network, that is, users and contracts. Accounts are used to identify the owner of trasactions, and specify the destinations of trasactions. All accounts can have tokens.

There are two types of accounts in ICON, accounts that are associated with users, often dubbed as Externally Owned Accounts (EOA), and Smart Contract (SCORE) Accounts. An EOA address starts with "hx" followed by a 20-byte hex string, while a Smart Contract Account address starts with "cx" followed by a 20-byte hex string. 

There are some special, pre-defined accounts, such as:
 - Governance contract
 - Public Treasury

In order to deploy a SCORE on the ICON network or make a transaction, one must hold a valid EOA. Furthermore, when we say account in this document, it means EOA. The terms of EOA, wallet, and keystore are not exactly same, but are often used interchangeably. 

An account is cryptographically defined by a private-public key pair, and the account address can be derived from its public key. Creating an account is equivalant to creating a key pair, and the account can be exported as a file known as a keystore file. 

## Keystore file
A keystore file is a JSON text file containing your private key and address. The private key is encrypted with the password that you enter when you create an account or keystore file. If you lose your keystore file and password, there is no way to recover them. You will lose you account and the assets you own.

A keystore file is used to authenticate a user. Every transaction must be signed with your private key. If your private key is leaked, anyone possessing the private key can access your account and sign the transaction on your behelf. Therefore, it is always recommended to use a keystore file instead of a plain private key. 


