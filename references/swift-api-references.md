---
title: "ICONKit, ICON SDK for Swift"
excerpt: ""
---

ICON supports SDK for 3rd party or user services development. You can integrate ICON SDK for your project and utilize ICON’s functionality.
This document provides you with an information of API specification.

# API Specification

## ICONService

`ICONService` is a class which provides APIs to communicate with ICON nodes. It enables you to easily use [ICON JSON-RPC APIs](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md) (version 3). All methods of `IconService` returns a `Request<T, ICError>` instance. 

### Initialize
```Swift
init(provider: String, nid: String)
```

#### Parameters
| Parameter | Type | Description |
| --------- | ---- | ----------- |
| provider | `String` | ICON node url. |
| nid | `String` | Network ID. |

For more details of node url and network id, see [ICON Network](https://github.com/icon-project/icon-project.github.io/blob/master/docs/icon_network.md) document.

#### Example

```Swift
// ICON Mainnet
let iconService = ICONService(provider: "https://ctz.solidwallet.io/api/v3", nid: "0x1")
```

## Queries

`Request` are executed as **synchronized** and **asynchronized**.

#### Synchronized execution

To execute the request synchronously, you need to run  `execute()`  function of `Request`.

```swift
public func execute() -> Result<T, ICError>
```

**Returns**

`Result<T, ICError>`

**Example**

```swift
// Synchronized request execution
let response = iconService.getLastBlock().execute()

switch response {
case .success(let block):
    print(block)
case .failure(let error):
    print(error.errorDescription)
}
```

#### Asynchronized execution

To execute the request asynchronously, you need to run  `async()`  function of `Request`.

```swift
public func async(_ completion: @escaping (Result<T, ICError>) -> Void)
```

**Returns**

`async()` is a closure. Passing `Result<T, ICError>`.

**Example**

```swift
// Asynchronized request execution
iconService.getLastBlock().async { (result) in
    switch result {
    case .success(let block):
      print(block)
    case .failure(let error):
      print(err.errorDescription)
    }
}
```

### List of Queries

- [getLastBlock](#getLastBlock)
- [getBlock](#getblock)
- [call](#call)
- [getBalance](#getBalance)
- [getScoreAPI](#getScoreAPI)
- [getTotalSupply](#getTotalSupply)
- [getTransaction](#getTransaction)
- [getTransactionResult](#getTransactionResult)


### getLastBlock

Get the last block information.

```Swift
func getLastBlock() -> Request<Response.Block>
```

#### Parameters

None

#### Returns

`Request` - Request instance for `icx_getLastBlock` JSON-RPC API request. If the execution is successful, returns `Result<Response.Block, ICError>`.

#### Example

```Swift
// Returns last block
let response = iconService.getLastBlock().execute()

switch response {
case .success(let result): // result == Response.Block
    print(result)
case .error(let error):
    print(error)
}
```

### getBlock

Get the block information.

```Swift
// Get block information by height.
func getBlock(height: UInt64) -> Request<Response.Block>

// Get block information by hash.
func getBlock(hash: String) -> Request<Resopnse.Block>
```

#### Parameters

| Parameter | Type     | Description           |
| --------- | -------- | --------------------- |
| height    | `UInt64` | Height value of block |
| hash      | `String` | Hash value of block   |

#### Returns

`Request` - Request instance for `icx_getBlockByHeight` | `icx_getBlockByHash`. If the execution is successful, returns `Result<Response.Block, ICError>`.

#### Example

```Swift
// Returns block information by block height
let heightBlock = iconService.getBlock(height: 10532).execute()

// Returns block information by block hash
let hashBlock = iconService.getBlock(hash: "0xdb310dd653b2573fd673ccc7489477a0b697333f77b3cb34a940db67b994fd95").execute()

switch heightBlock {
case .success(let block): // result == Response.Block
    print(block)
case .error(let error):
    print(error)
}
```

### call

Calls external function of SCORE API.

```Swift
func call<T>(_ call: Call<T>) -> Request<T>
```

#### Parameters

| Parameter | Type   | Description                |
| --------- | ------ | -------------------------- |
| call      | `Call` | an instance of Call class. |

#### Returns

`Request<T>` - Request instance for `icx_call` JSON-RPC request. If the execution is successful, returns `Result<T, ICError>`.

#### Example

In `Call` generic value, input the type of function call. The generic value should follow Codable protocol. 

For example, the type of SCORE function call is `String`, input `String ` like this. 

```swift
// Create Call instance 
let call = Call<String>(from: wallet.address, to: "cxf9148db4f8ec78823a50cb06c4fed83660af38d0", method: "tokenOwner", params: nil)

let request: Request<String> = iconService.call(call)
let response: Result<String, ICError> = request.execute()

switch response {
case .success(let owner):
	 print(owner) // hx70b591822468415e86fb9ba1354214902ea76196
case .failure(let error):
	print(error)
}
```

If SCORE call result is hex `String` , you can choice return types `String` or `BigUInt`. 

Using `Call<String>`, return original hex String value. 

Using `Call<BigUInt>` return converted BigUInt value.

```Swift
let ownerAddress: String = "hx07abc7a5b8a4941fc0b6930c88b462995acf929b"
let scoreAddress: String = "cxf9148db4f8ec78823a50cb06c4fed83660af38d0"

// Using `String`. Return hex String.
let call = Call<String>(from: ownerAddress, to: scoreAddress, method: "balanceOf", params: ["_owner": ownerAddress])
let request: Request<String> = iconService.call(call)
let response: Result<String, ICError> = request.execute()

switch response {
case .success(let balance):
	 print("balance: \(balance)") // "balance: 0x11e468ef422cc60031ef"
case .failure(let error):
	print(error)
}


// Using `BigUInt`. Return BigUInt.
let call = Call<BigUInt>(from: ownerAddress, to: scoreAddress, method: "balanceOf", params: ["_owner": ownerAddress])
let request: Request<BigUInt> = iconService.call(call)
let response: Result<BigUInt, ICError> = request.execute()

switch response {
case .success(let balance):
	 print("balance: \(balance)") // "balance: 84493649192649192649199"
case .failure(let error):
	print(error)
}
```

### getBalance

Get the balance of the address.

```Swift
func getBalance(address: String) -> Request<BigUInt>
```

#### Parameters

| Parameter | Type     | Description    |
| --------- | -------- | -------------- |
| address   | `String` | An EOA Address |

#### Returns

`Request` - Request instance for `icx_getBalance` JSON-RPC API request. If the execution is successful, returns `Result<BigUInt, ICError>`.

#### Example

```Swift
// Returns the balance of specific address
let response = iconService.getBalance(address: "hx9d8a8376e7db9f00478feb9a46f44f0d051aab57").execute()

switch response {
case .success(let result): // result == BigUInt
    print(result)
case .error(let error):
    print(error)
}
```

### getScoreAPI

Get the SCORE API list.

```Swift
func getScoreAPI(scoreAddress: String) -> Request<[Response.ScoreAPI]>
```

#### Parameters

| Parameter    | Type     | Description     |
| ------------ | -------- | --------------- |
| scoreAddress | `String` | A SCORE address |

#### Returns

`Request` - Request instance for `icx_getScoreApi` JSON-RPC API request. If the execution is successful, returns `Result<[Response.ScoreAPI], ICError>`.

#### Example

```Swift
// Returns the SCORE API list
let response = iconService.getScoreAPI(scoreAddress: "cx0000000000000000000000000000000000000001").execute()

switch response {
case .success(let result): // result == [Response.ScoreAPI]
    print(result)
case .error(let error):
    print(error)
}
```

### getTotalSupply

get the total number of issued coins.
```Swift
func getTotalSupply() -> Request<BigUInt>
```

#### Parameters

None

#### Returns

`Request` - Request instance for `icx_getTotalSupply` JSON-RPC API request. If the execution is successful, returns `Result<BigUInt, ICError>`.

#### Example
```Swift
// Returns total value of issued coins
let response = iconService.getTotalSupply().execute()

switch response {
case .success(let result): // result == BigUInt
    print(result)
case .error(let error):
    print(error)
}
```

### getTransaction
Get the transaction information.
```Swift
func getTransaction(hash: String) -> Request<Response.TransactionByHashResult>
```

#### Parameters
| Parameter | Type | Description |
| --- | --- | --- |
| hash | `String` | Hash value of transaction |

#### Returns
`Request` - Request instnace for `icx_getTransactionByHash` JSON-RPC API request. If the execution is successful, returns `Result<Response.TransactionByHashResult, ICError>`.

#### Example
```Swift
// Returns transaction information
let response = iconService.getTransaction(hash: "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238").execute()

switch response {
case .success(let result): // result == Response.TransactionByHashResult
    print(result)
case .error(let error):
    print(error)
}
```

### getTransactionResult
Get the transaction result information.
```Swift
func getTransactionResult(hash: String) -> Request<Response.TransactionResult>
```

#### Parameters
| Parameter | Type | Description |
| --- | --- | --- |
| hash | `String` | A transaction hash. |

#### Returns
`Request` - Request instance for `icx_getTransactionResult` JSON-RPC API request. If the execution is successful, returns `Result<Response.TransactionResult, ICError>`.

#### Example
```Swift
// Returns transaction result information
let response = iconService.getTransactionResult(hash: "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238").execute()

switch response {
case .success(let result): // result == Response.TransactionResult
    print(result)
case .error(let error):
    print(error)
}
```

## Wallet

```Swift
class Wallet
```
A class which provides EOA funcstions. It enables you to create, transform to Keystore or load wallet from Keystore.

### Initialize
```Swift
init(privateKey: PrivateKey?)
```

#### Parameters 
| Parameter | Type | Description |
| --- | --- | --- |
| privateKey | `PrivateKey` | If privateKey is null, it generates new private key while initializing. |

#### Returns
`Wallet` - Wallet instance.

#### Example
```Swift
// Create new wallet
let wallet = Wallet(prviateKey: nil)    // will generate private key

// Imports from exist private key
let privateKey = PrivateKey(hexData: privateKeyData)
let wallet = Wallet(privateKey: privateKey)
```

### Loading wallets and storing the Keystore

```swift
// Save wallet keystore.
let wallet = Wallet(privateKey: nil)
do {
    try wallet.generateKeystore(password: "YOUR_WALLET_PASSWORD")
    try wallet.save(filepath: "YOUR_STORAGE_PATH")
} catch {
    // handle errors
}

// Load a wallet from the keystore.
do {
    let jsonData: Data = try Data(contentsOf: "YOUR_KEYSTORE_PATH")
    let decoder = JSONDecoder()
    let keystore = try decoder.decoder(Keystore.self, from: jsonData)
    let wallet = Wallet(keystore: keystore, password: "YOUR_WALLET_PASSWORD")
} catch {
    // handle errors
}
```

### getSignature

Generate signature string by signing transaction data.
```Swift
func getSignature(data: Data) throws -> String
```

#### Parameter
| Parameter | Type | Description |
| --- | --- | --- |
| data | `Data` | the serialized transaction data. |

#### Returns
`String` - A signature string.(**Base64 encoded**)

#### Example
```Swift
let wallet = Wallet(privateKey: nil)
do {
    let signature = try wallet.getSignature(data: toBeSigned)
    print("signature - \(signature)")
} catch {
    // handle error
}
```

### Properties

| Property | Type | Description |
| --- | --- | --- |
| address | `String` | Return wallet's address (read-only) |

## Transactions

`Transaction` class is enable you to make transaction instance. There are 3 types of Transaction class.

| Class                                     | Description                                                  |
| ----------------------------------------- | ------------------------------------------------------------ |
| [Transaction](#Transaction)               | A class for Transaction instance which is for sending ICX.   |
| [CallTransaction](#CallTransaction)       | A class for CallTransaction instance which is for SCORE function call. CallTransaction extends `Transaction` class. |
| [MessageTransaction](#MessageTransaction) | A class for MessageTransaction instance which is for transfer message. Extends `Transaction` class. |

### Transaction

```Swift
class Transaction
```
`Transaction` is a class representing a transaction data used for sending ICX.

#### Initialize

```Swift
// Create empty transaction object
init()
// Create transaction with data
convenience init(from: String, to: String, stepLimit: BigUInt, nid: String, value: BigUInt, nonce: String, dataType: String? = nil, data: Any? = nil)
```

#### Parameters

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| from      | An EOA address that creates the transaction.                 |
| to        | An EOA address to receive coins or SCORE address to execute the transaction. |
| stepLimit | Amounts of Step limit.                                       |
| nid       | A network ID.                                                |
| value     | Sending amount of ICX in loop unit.                          |
| nonce     | An arbitrary number used to prevent transaction hash collision. |

#### from

Setter for `from` property.
```Swift
func from(_ from: String) -> Self
```

**Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| from | `String` | An EOA address that creates the transaction. |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
// Set from property
let transaction = Transaction()
		.from("hx9043346dbaa72bca42ecec6b6e22845a4047426d")
```

#### to
Setter for `to` property.
```Swift
func to(_ to: String) -> Self
```

**Paramters**

| Paramter | Type | Description |
| --- | --- | --- |
| to | `String` | An EOA or SCORE address. |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
		.to("hx2e26d96bd7f1f46aac030725d1e302cf91420458")
```

#### stepLimit
Setter for `setpLimit` property.
```Swift
func stepLimit(_ limit: BigUInt) -> Self
```

**Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| limit | `BigUInt` | Amounts of Step limit |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
		.stepLimit(BigUInt(1000000))
```

#### nid
Setter for `nid` property.
```Swift
func nid(_ nid: String) -> Self
```

**Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| nid | `String` | A network ID. (Mainnet = "0x1", Testnet = "0x2", [etc](https://github.com/icon-project/icon-project.github.io/blob/master/docs/icon_network.md)) |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
		.nid("0x1")
```

#### value
Setter for `value` property.
```Swift
func value(_ value: BigUInt) -> Self
```

**Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| value | `BigUInt` | Sending amount of ICX in loop unit. (1 ICX = 10^18^ loop) |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
    .value(BigUInt(15000000))
```

#### nonce
Setter for `nonce` property.
```Swift
func nonce(_ nonce: String) -> Self
```

**Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| nonce | `String` | A nonce value. |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
    .nonce("0x1")
```

#### Example

```swift
// Creating transaction instance for Sending ICX.
let transaction = Transaction()
    .from("hx9043346dbaa72bca42ecec6b6e22845a4047426d")
    .to("hx2e26d96bd7f1f46aac030725d1e302cf91420458")
    .value(BigUInt(15000000))
    .stepLimit(BigUInt(1000000))
    .nid("0x1")
    .nonce("0x1")
```

### CallTransaction

```swift
class CallTransaction: Transaction
```

`CallTransaction` class is used for invoking a state-transition function of SCORE. It extends `Transaction` class, so instance parameters and methods of class are mostly identical to `Transaction` class, except for the following:

#### Parameters

| Parameters | Description                   |
| ---------- | ----------------------------- |
| method     | The method name of SCORE API. |
| params     | The input params for method.  |

For details of extended parameters and methods, see [Transaction](#Transaction) section.

#### method

Transaction for invoking a *state-transition* function of SCORE.

*Transaction parameter `dataType` will be fixed with `call`*

```Swift
func method(_ method: String) -> Self
```

**Parameters**

| Parameter | Type     | Description                   |
| --------- | -------- | ----------------------------- |
| method    | `String` | the method name of SCORE API. |

**Returns**

Returns `CallTransaction` itself.

#### params

The parameters for SCORE method used for `call` function.

```swift
func params(_ params: [String: Any]) -> Self
```

**Parameter**

| Parameter | Type                      | Description                       |
| --------- | ------------------------- | --------------------------------- |
| params    | `Dictionary<String: Any>` | Input parameters for call method. |

**Returns**

Returns `CallTransaction` itself.

#### Example

```swift
// Creating transaction instance for SCORE function call
let call = CallTransaction()
    .from(wallet.address)
    .to("cx000...001")
    .stepLimit(BigUInt(1000000))
    .nid(self.iconService.nid)
    .nonce("0x1")
    .method("transfer")
    .params(["_to": to, "_value": "0x1234"])
```

### MessageTransaction

```swift
class MessageTransaction: Transaction
```

`MessageTransaction` class is used for sending message data. It extends `Transaction` class, so instance parameters and methods of class are mostly identical to `Transaction` class, except for the following:

#### Parameters

| Parameters | Description        |
| ---------- | ------------------ |
| message    | A message to send. |

For details of extended parameters and methods, see [Transaction](#Transaction) section.

#### message

Send messages.

*Transaction parameter `dataType` will be fixed with `message`.*

```Swift
func message(_ message: String) -> Self
```

**Parameters**

| Parameter | Type | Description |
| --- | --- | --- |
| message | `String` | A message String. |

**Returns**

Returns `MessageTransaction` itself.

#### Example

```swift
// Creating transaction instance for transfering message.
let messageTransaction = MessageTransaction()
    .from("hx9043346dbaa72bca42ecec6b6e22845a4047426d")
    .to("hx2e26d96bd7f1f46aac030725d1e302cf91420458")
    .value(BigUInt(15000000))
    .stepLimit(BigUInt(1000000))
    .nonce("0x1")
    .nid("0x1")
    .message("Hello, ICON!")
```

### SignedTransaction

`SignedTransaction` is a class for signing transaction object. It enables you to make signed transaction.

#### Initialize

```swift
init(transaction: Transaction, privateKey: PrivateKey)
```

**parameters**

| Parameter   | Type          | Description                        |
| ----------- | ------------- | ---------------------------------- |
| transaction | `Transaction` | A transaction that will be signed. |
| privateKey  | `PrivateKey`  | A privateKey.                      |

#### Example

```swift
let signed = try SignedTransaction(transaction: transaction, privateKey: yourPrivateKey)
```

### sendTransaction

Send a transaction that changes the state of address. Request is executed as **Synchronized** or **Asynchronized** like a querying request.

```Swift
func sendTransaction(signedTransaction: SignedTransaction) -> Request<String>
```

#### Parameters

| Parameter         | Type                | Description                                                  |
| ----------------- | ------------------- | ------------------------------------------------------------ |
| signedTransaction | `SignedTransaction` | an instance of [SignedTransaction](#SignedTransaction) class. |

#### Returns

`Request` - Request instance for `icx_sendTransaction` JSON-RPC request. If the execution is successful, returns `Result<String, ICError>`.

#### Example

```Swift
// Synchronized request
let response = iconService.sendTransaction(signedTransaction: signed).execute()

switch response {
case .success(let result): // result == String
    print(result)
case .error(let error):
    print(error)
}

// Asynchronized request
let request = iconService.sendTransaction(signedTransaction: signed)
request.async { (result) in
    switch result {
    case .success(let result): // result == String
        print(result)
    case .failure(let error):
        print(error)
    }
}
```

## ICError

There are 5 types of errors.

- **emptyKeystore** - Keystore is empty. 

- **invalid**
  - missing(parameter: JSONParameterKey) - Missing JSON parameter.
  - malformedKeystore - Keystore data malformed.
  - wrongPassword - Wrong password.

- **fail**
  - sign - Failed to signing.
  - parsing - Failed to parsing.
  - decrypt - Failed to decrypt.
  - convert - Failed to convert to url or data.

- **error(error: Error)**

- **message(error: String**) -  [JSON-rpc-Error Messages](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md#error-codes)

## Converters

ICONKit supports converter functions.

### convert()

Convert ICX or gLoop to loop.

> 1 ICX = 10 ^9^ gLoop = 10 ^18^ loop

#### Paramters

| Parameters | Type   | Description                                                  |
| ---------- | ------ | ------------------------------------------------------------ |
| unit       | `Unit` | The unit of value( `.icx`, `.gLoop`, `.loop` ). Default value is  `.icx`. |

#### Returns

`BigUInt` - The value that converted to loop.

#### Example

```swift
let balance: BigUInt = 100

// Convert ICX to loop.
let ICXToLoop: BigUInt = balance.convert() // 100000000000000000000

// Convert gLoop to loop
let gLoopToLoop: BigUInt = balance.convert(unit: .gLoop) // 100000000000
```

### toHexString()

Convert `BigUInt` value to hex `String`.

```swift
public func toHexString(unit: Unit = .loop) -> String 
```

#### Parameters

| Parameters | Type   | Description                                                  |
| ---------- | ------ | ------------------------------------------------------------ |
| unit       | `Unit` | The unit of value( `.icx`, `.gLoop`, `.loop` ). Default value is  `.loop`. |

#### Returns

`String` - The value converted to a hex String.

#### Example

```swift
// Convert `BigUInt` value to hex `String`.
let ICXToLoop: BigUInt = 100000000000000000000
let hexString: String = ICXToLoop.toHexString() // 0x56bc75e2d63100000
```

### hexToBigUInt()

Convert hex `String` to `BigUInt`.

```swift
public func hexToBigUInt(unit: Unit = .loop) -> BigUInt?
```

#### Parameters

| Parameters | Type   | Description                                                  |
| ---------- | ------ | ------------------------------------------------------------ |
| unit       | `Unit` | The unit of value( `.icx`, `.gLoop`, `.loop` ). Default value is  `.loop`. |

#### Returns

`BigUInt`  - The value that converted to hex String.

If the conversion is failed, return `nil`.

#### Example

```swift
// Convert hex `String` to `BigUInt`.
let hexString: String = "0x56bc75e2d63100000"
let hexBigUInt: BigUInt = hexString.hexToBigUInt()! // 100000000000000000000
```

## References

- [ICON JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md)

- [ICON Network](https://github.com/icon-project/icon-project.github.io/blob/master/docs/icon_network.md)

