---
title: "ICONKit, ICON sdk for Swift"
excerpt: ""
---

ICON supports SDK for 3rd party or user services development. You can integrate ICON SDK for your project and utilize ICON’s functionality.
This document provides you with an information of installation and API specification.

## Installation

### CocoaPods
[CocoaPods](https://cocoapods.org/) is a dependecy manager for Swift Cocoa projects.
```
$ sudo gem install cocoapods
```

To integrate ICONKit into your project, specify it in your `Podfile`

```
target '<Your Target Name>' do
    use_frameworks!
    ...
    pod 'ICONKit', '~> 0.2.5'
    ...
end
```

Now install the ICONKit
```
$ pod install
```

## Result
ICONKit uses [Result](https://github.com/antitypical/Result) framework. All functions of ICONService returns Result<T, ICONResult>. `T` for successor, `ICONResult` for error.
Refer to [Result](https://github.com/antitypical/Result) for more detail.

## Quick start
A simple query of the block by height is as follows.
```Swift
let service = ICONService(provider: "https://ctz.solidwallet.io/api/v3", nid: "0x1")

// Gets a block matching the block height.
let request: Request<Response.Block> = service.getBlock(height: height)
let result = request.execute()

switch result {
case: .success(let responseBlock):
    ...
case: .failure(let error):
    print(error)
}
```
## ICONService
APIs called through `ICONService`.

It can be initialized as follows.
```Swift
let iconService = ICONService(provider: "https://ctz.solidwallet.io/api/v3", nid: "0x1")
```

## Queries
All queries are requested by a `Request<T>`.

Its requests are executed as **Synchronized**.
(*will supports asynchronized version soon.*)

```Swift
let response = service.getBlock(height: height).execute()

switch response {
case: .success(let responseBlock):
    ...
case: .failure(let error):
    print(error)
}
```
The querying APIs are as follows.
```Swift
// Gets the block
let request:  = service.getBlock(height: height)
let request: Request<Response.Block> = service.getBlock(hash: "0x000...000")
let request: Request<Response.Block> = service.getLastBlock()

// Gets the balance of an given account
let request: Request<Response.IntValue> = service.getBalance(address: "hx000...1")

// Gets a list of ScoreAPI
let request: Request<Response.ScoreAPI> = service.getScoreAPI(scoreAddress: "cx000...1")

// Gets the total supply of ICX
let request: Request<Response.IntValue> = service.getTotalSupply()

// Gets a transaction matching the given transaction hash
let request: Request<Response.Transaction> = service.getTransaction(hash: "0x000...000")

// Gets the result of the transaction matching the given transaction hash
let request: Request<Response.TransactionResult> = service.getTransactionResult(hash: "0x000...000")

// Calls a SCORE API just for reading
let call = Call(from: wallet.address, to: scoreAddress, method: "balanceOf", params: params)
let request: Request<Response.IntValue> = service.call(call)
```
## Sending transactions
Calling SCORE APIs to change states is requested as sending a transaction.

Before sending a transaction, the transaction should be signed.

**Loading wallets and storing the Keystore**
```Swift
// Generates a wallet.
let wallet = Wallet(privateKey: nil)

// OR

// Load a wallet from the private key.
let privateKey = PrivateKey(hexData: data)
let wallet = Wallet(privateKey: privateKey)
```

**Creating transactions**

```Swift
// Sending ICX
let transaction = Transaction()
    .from(wallet.address)
    .to(to)
    .value(BigUInt(15000000))
    .stepLimit(BigUInt(1000000))
    .nonce("0x1")
    .nid(service.nid)

// Call
let transaction = Transaction()
    .from(wallet.address)
    .to(scoreAddress)
    .stepLimit(BigUInt(1000000))
    .call("transfer")
    .nonce("0x1")
    .nid(service.nid)
    .params(["_to": to, "_value": "0x1234"])

// Message
let transaction = Transaction()
    .from(wallet.address)
    .to(to)
    .value(BigUInt(15000000))
    .stepLimit(BigUInt(1000000))
    .message("Hello, World!")
    .nonce("0x1")
    .nid(service.nid)
```
`SignedTransaction` object signs a transaction using the wallet.

```Swift
do {
    let signed = try SignedTransaction(transaction: transaction, privateKey: privateKey)

    let request: Request<Response.TxHash> = service.sendTransaction(signedTransaction: signed)
    let response = request.execute()
    switch response {
    case .success(let result):
        print("tx result - \(result)")

    case .failure(let error):
        // Error handling
        print("error")
    }
    ...
} catch {
    print(error)
    ...
}
```
## Reference
- [ICON JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md)
- [ICON Network](https://github.com/icon-project/icon-project.github.io/blob/master/docs/icon_network.md)

## Version
0.2.4 (Beta)

## Requirement
- Xcode 10 or higher
- iOS 10 or higher
- Swift 4
- [Cocoapods](https://cocoapods.org)

## License

This project follows the Apache 2.0 License. Please refer to [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) for details.
