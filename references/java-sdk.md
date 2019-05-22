---
title: "Java SDK"
excerpt: "The source code is found on Github at https://github.com/icon-project/icon-sdk-java"
---

This document describes how to interact with `ICON Network` using Java SDK. This document contains SDK installation, API usage guide, and code examples.

Get different types of examples as follows.

| Code Example                                            | Description                                                  |
| ------------------------------------------------------- | ------------------------------------------------------------ |
| [Wallet](#section-wallet)                                       | An example of creating and loading a keywallet.              |
| [ICX Transfer](#section-icx-transfer)                           | An example of transferring ICX and confirming the result.    |
| [Token Deploy and Transfer](#section-token-deploy-and-transfer) | An example of deploying an IRC token, transferring the token and confirming the result. |
| [Sync Block](#section-sync-block)                               | An example of checking block confirmation and printing the ICX and token transfer information. |

This document is focused on how to use the SDK properly. For the detailed API specification, see the API reference documentation.

## Prerequisite

This Java SDK works on the following platforms:

- Java 8+ (for Java7, you can explore source code [here](https://github.com/icon-project/icon-sdk-java/blob/sdk-for-java7/README.md))
- Android 3.0+ (API 11+)

## Installation

Download [the latest JAR](https://search.maven.org/search?q=g:foundation.icon%20a:icon-sdk) or grab via Maven:

```xml
<dependency>
    <groupId>foundation.icon</groupId>
    <artifactId>icon-sdk</artifactId>
    <version>[x.y.z]</version>
</dependency>
```

or Gradle:

```groovy
dependencies {
    implementation 'foundation.icon:icon-sdk:[x.y.z]'
}
```

## Using the SDK

### IconService

APIs are called through `IconService`.
`IconService` can be initialized as follows.

```java
// Creates an instance of IconService using the HTTP provider.
IconService iconService = new IconService(new HttpProvider("http://localhost:9000", 3));
```

The code below shows initializing `IconService` with a custom HTTP client.

```java
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .readTimeout(200, TimeUnit.MILLISECONDS)
    .writeTimeout(600, TimeUnit.MILLISECONDS)
    .build();

IconService iconService = new IconService(new HttpProvider(okHttpClient, "http://localhost:9000", 3));
```

### Queries

All queries are requested by a `Request` object.
Query requests can be executed as **Synchronized** or **Asynchronized**.
Once the request has been executed, the same request object cannot be executed again.

```java
Request<Block> request = iconService.getBlock(height);

// Asynchronized request execution
request.execute(new Callback<Block>(){
    void onFailure(Exception exception) {
        ...
    }

    void onResponse(Block block) {
        ...
    }
});

// Synchronized request execution
try {
    Block block = request.execute();
    ...
} catch (Exception e) {
    ...
}
```

The querying APIs are as follows.

```java
// Gets the block
Request<Block> request = iconService.getBlock(new BigInteger("1000")); // by height
Request<Block> request = iconService.getBlock(new Bytes("0x000...000"); // by hash
Request<Block> request = iconService.getLastBlock(); // the last block

// Gets the balance of an given account
Request<BigInteger> request = iconService.getBalance(new Address("hx000...1");

// Gets a list of the SCORE API
Request<List<ScoreApi>> request = iconService.getScoreApi(new Address("cx000...1"));

// Gets the total supply of icx
Request<BigInteger> request = iconService.getTotalSupply();

// Gets a transaction matching the given transaction hash
Request<Transaction> request = iconService.getTransaction(new Bytes("0x000...000"));

// Gets the result of the transaction matching the given transaction hash
Request<TransactionResult> request = iconService.getTransactionResult(new Bytes("0x000...000"));

// Calls a SCORE read-only API
Call<BigInteger> call = new Call.Builder()
    .from(wallet.getAddress())
    .to(scoreAddress)
    .method("balanceOf")
    .params(params)
    .buildWith(BigInteger.class);
Request<BigInteger> request = iconService.call(call);

// Calls without response type
Call<RpcItem> call = new Call.Builder()
    .from(wallet.getAddress())
    .to(scoreAddress)
    .method("balanceOf")
    .params(params)
    .build();
Request<RpcItem> request = iconService.call(call);
try {
    RpcItem rpcItem = request.execute();
    BigInteger balance = rpcItem.asInteger();
    ...
} catch (Exception e) {
    ...
}
```

### Transactions

Calling SCORE APIs to change states is requested as sending a transaction.

Before sending a transaction, the transaction should be signed. It can be done using a `Wallet` object.

**Loading wallets and storing the Keystore**

```java
// Generates a wallet.
Wallet wallet = KeyWallet.create();

// Loads a wallet from the private key.
Wallet wallet = KeyWallet.load(new Bytes("0x0000"));

// Loads a wallet from the key store file.
File file = new File("./key.keystore");
Wallet wallet = KeyWallet.load("password", file);

// Stores the keystore on the file path.
File dir = new File("./");
KeyWallet.store(wallet, "password", dir); // throw exception if an error exists.
```

**Creating transactions**

```java
// send ICX
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(wallet.getAddress())
    .to(scoreAddress)
    .value(new BigInteger("150000000"))
    .stepLimit(new BigInteger("1000000"))
    .nonce(new BigInteger("1000000"))
    .build();

// deploy a SCORE
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(wallet.getAddress())
    .to(scoreAddress)
    .stepLimit(new BigInteger("5000000"))
    .nonce(new BigInteger("1000000"))
    .deploy("application/zip", content)
    .params(params)
    .build();

// call a method in SCORE
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(wallet.getAddress())
    .to(scoreAddress)
    .value(new BigInteger("150000000"))
    .stepLimit(new BigInteger("1000000"))
    .nonce(new BigInteger("1000000"))
    .call("transfer")
    .params(params)
    .build();

// send a message
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(wallet.getAddress())
    .to(scoreAddress)
    .value(new BigInteger("150000000"))
    .stepLimit(new BigInteger("1000000"))
    .nonce(new BigInteger("1000000"))
    .message(message)
    .build();
```

`SignedTransaction` object signs a transaction using the wallet.

And the request can be executed as **Synchronized** or **Asynchronized** like a query request.

Once the request has been executed, the same request object cannot be executed again.

```java
SignedTransaction signedTransaction = new SignedTransaction(transaction, wallet);

Request<Bytes> request = iconService.sendTransaction(signedTransaction);

// Asynchronized request execution
request.execute(new Callback<Bytes>(){
    void onFailure(Exception e) {
        ...
    }

    void onResponse(Bytes txHash) {
        ...
    }
});

// Synchronized request execution
try {
    Bytes txHash = request.execute();
    ...
} catch (Exception e) {
    ...
}
```


### Step Estimation

It is important to set a proper `stepLimit` value in your transaction to make the submitted transaction executed successfully.

`estimateStep` API provides a way to **estimate** the Step usage of a given transaction. Using the method, you can get an estimated Step usage before sending your transaction then make a `SignedTransaction` with the `stepLimit` based on the estimation.

```java
// make a raw transaction without the stepLimit
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(fromAddress)
    .to(toAddress)
    .nonce(BigInteger.valueOf(1))
    .call("transfer")
    .params(params)
    .build();

// get an estimated step value
BigInteger estimatedStep = iconService.estimateStep(transaction).execute();

// set some margin value
BigInteger margin = BigInteger.valueOf(10000);

// make a signed transaction with the same raw transaction and the estimated step
SignedTransaction signedTransaction = new SignedTransaction(transaction, wallet, estimatedStep.add(margin));
Bytes txHash = iconService.sendTransaction(signedTransaction).execute();
...
```

Note that the estimation can be smaller or larger than the actual amount of step to be used by the transaction,
so it is recommended to add some margin to the estimation when you set the `stepLimit` of the `SignedTransaction`.

### Converter

All the requests and responses values are parcelled as `RpcItem` (`RpcObject`, `RpcArray`, `RcpValue`). You can convert your own class using `RpcConverter`.

```java
iconService.addConverterFactory(new RpcConverter.RpcConverterFactory() {
    @Override
    public  RpcConverter create(Class type) {
        if (type.isAssignableFrom(Person.class)) {
            return new RpcConverter<Person>() {
                @Override
                public Person convertTo(RpcItem object) {
                    // Unpacking from RpcItem to the user defined class
                    String name = object.asObject().getItem("name").asString();
                    BigInteger age = object.asObject().getItem("age").asInteger();
                    return new Person(name, age);
                }

                @Override
                public RpcItem convertFrom(Person person) {
                    // Packing from the user defined class to RpcItem
                    return new RpcObject.Builder()
                        .put("name", person.name)
                        .put("age", person.age)
                        .build();
                }
            };
        }
        return null;
    }
});

...

class Person {
    public Person(String name, BigInteger age) {}
}

...

Call<Person> call = new Call.Builder()
    .from(fromAddress)
    .to(scoreAddress)
    .method("searchMember")
    .params(person) // the input parameter is an instance of Person type
    .buildWith(Person.class); // build with the response type 'Person'

Person memberPerson = iconService.call(call).execute();
```



## Code Examples

### Wallet

This example shows how to create a new `KeyWallet` or load wallet with a private key or Keystore file.

#### Create a wallet

Create new EOA by calling `create` function. After creation, the address and private key can be looked up.

```java
KeyWallet wallet = KeyWallet.create(); // Wallet Creation
System.out.println("address:" + wallet.getAddress()); // Address Check
System.out.println("privateKey:" + wallet.getPrivateKey()); // PrivateKey Check

// Output
address:hx4d37a7013c14bedeedbe131c72e97ab337aea159
privateKey:00e1d6541bfd8be7d88be0d24516556a34ab477788022fa07b4a6c1d862c4de516
```

#### Load a wallet

You can call existing EOA by calling `load` function.

After creation, address and private key can be looked up.

```java
String privateKey;
KeyWallet wallet = KeyWallet.load(new Bytes(privateKey));  // Load keyWallet with privateKey
System.out.println("address:" + wallet.getAddress()); // Address lookup
System.out.println("privateKey:" + wallet.getPrivateKey()); // PrivateKey lookup
```

#### Store the wallet

After `KeyWallet` object creation, the Keystore file can be stored by calling `store` function.

After calling `store`, the Keystore file’s name can be looked up with the returned value.

```java
String password;
KeyWallet wallet; /* create or load keywallet */
File destinationDirectory = new File(/* directory Path */);
String fileName = KeyWallet.store(wallet, password, destinationDirectory);
System.out.println("fileName:" + fileName); // Keystore file name output

// Output
fileName:UTC--2018-08-30T03-27-41.768000000Z--hx4d37a7013c14bedeedbe131c72e97ab337aea159.json
```

### ICX Transfer

This example shows how to transfer ICX and check the result.

*For the KeyWallet and IconService creation, please refer to the information above.*

#### ICX transfer transaction

In this example, you can create a KeyWallet with `CommonData.PRIVATE_KEY_STRING` and transfer 1 ICX to `CommonData.ADDRESS_1`.

```java
Wallet wallet = KeyWallet.load(new Bytes(CommonData.PRIVATE_KEY_STRING));
Address toAddress = new Address(CommonData.ADDRESS_1);
// 1 ICX -> 1000000000000000000 loop conversion
BigInteger value = IconAmount.of("1", IconAmount.Unit.ICX).toLoop();
```

Generate a transaction using the values above.

```java
// Network ID ("1" for Mainnet, "2" for Testnet, etc)
BigInteger networkId = new BigInteger("3");
// Only `default` step cost is required to transfer ICX: it is `100000` as of now.
BigInteger stepLimit = new BigInteger("100000");

// Timestamp is used to prevent the identical transactions. Only current time is required (Standard unit: us)
// If the timestamp is considerably different from the current time, the transaction will be rejected.
long timestamp = System.currentTimeMillis() * 1000L;

//Enter transaction information
Transaction transaction = TransactionBuilder.newBuilder()
        .nid(networkId)
        .from(fromAddress)
        .to(toAddress)
        .value(value)
        .stepLimit(stepLimit)
        .timestamp(new BigInteger(Long.toString(timestamp)))
        .build();
```

Generate a `SignedTransaction` to add the signature of the transaction.

```java
// Create signature of the transaction
SignedTransaction signedTransaction = new SignedTransaction(transaction, wallet);
// Read params to transfer to nodes
System.out.println(signedTransaction.getProperties());
// Send the transaction
Bytes txHash = iconService.sendTransaction(signedTransaction).execute();
```

#### Check the transaction result

After a transaction is sent, the result can be looked up with the returned hash value.

In this example, you can check your transaction result in every 2 seconds because of the block confirmation time. Checking the result is as follows:

```java
// Check the result with the transaction hash
TransactionResult result = iconService.getTransactionResult(hash).execute();
System.out.println("transaction status(1:success, 0:failure):"+result.getStatus());

// Output
transaction status(1:success, 0:failure):1
```

You can check the following information using the TransactionResult.

- status : 1 (success), 0 (failure)
- to : transaction’s receiving address
- failure : Only exists if the status is 0(failure). code(str), message(str) property included
- txHash : transaction hash
- txIndex : transaction index in a block
- blockHeight : Block height of the transaction
- blockHash : Block hash of the transaction
- cumulativeStepUsed : Accumulated amount of consumed step until the transaction is executed in block
- stepUsed : Consumed step amount to send the transaction
- stepPrice : Consumed step price to send the transaction
- scoreAddress : SCORE address if the transaction generated SCORE (optional)
- eventLogs : List of EventLogs written during the execution of the transaction.
- logsBloom : Bloom Filter of the indexed data of the Eventlogs.

#### Check the ICX balance

In this example, you can check the ICX balance by looking up the transaction before and after the transaction.

ICX balance can be confirmed by calling `getBalance` function from `IconService`

```java
KeyWallet wallet; /* create or load */
// Check the wallet balance
BigInteger balance = iconService.getBalance(wallet.getAddress()).execute();
System.out.println("balance:" + balance));

// Output
balance:5000000000000000000
```

### Token Deploy and Transfer

This example shows how to deploy a token and check the result. After that, shows how to send tokens and check the balance.

*For KeyWallet and IconService generation, please refer to the information above.*

#### Token deploy transaction

You need a SCORE project to deploy token.

In this example, you will use ‘sampleToken.zip’ from the ‘resources’ folder.

* sampleToken.zip: SampleToken SCORE project zip file.

Generate a keyWallet using `CommonData.PRIVATE_KEY_STRING`, then read the binary data from ‘sampleToken.zip’

```java
Wallet wallet = KeyWallet.load(new Bytes(CommonData.PRIVATE_KEY_STRING));
// Read sampleToken.zip from ‘resources’ folder.
byte[] content = /* score binary */
```

Prepare basic information of the token you want to deploy and make a parameter object.

```java
BigInteger initialSupply = new BigInteger("100000000000");
BigInteger decimals = new BigInteger("18");

RpcObject params = new RpcObject.Builder()
	.put("_initialSupply", new RpcValue(initialSupply))
	.put("_decimals", new RpcValue(decimals))
	.build();
```

Generate a raw transaction to deploy a token SCORE without the stepLimit value.

```java
Transaction transaction = TransactionBuilder.newBuilder()
	.nid(networkId)
	.from(wallet.getAddress())
	.to(CommonData.SCORE_INSTALL_ADDRESS)
	.timestamp(new BigInteger(Long.toString(timestamp)))
	.deploy(contentType, content)
	.params(params)
	.build();
```

Get an estimated Step value using `estimateStep` API of `IconService`.

```java
BigInteger estimatedStep = iconService.estimateStep(transaction).execute();
```

Generate a `SignedTransaction` with the same raw transaction and the estimated Step.
Note that the estimation can be smaller or larger than the actual amount of step to be used by the transaction.
So we need to add some margin value to the estimation when you set `stepLimit` parameter of `SignedTransaction`.

```java
// Set some margin value for the operation of `on_install`
BigInteger margin = BigInteger.valueOf(10000);

SignedTransaction signedTransaction = new SignedTransaction(transaction, wallet, estimatedStep.add(margin));
```

Calling `sendTransaction` API of `IconService` will return the transaction hash.

```java
Bytes txHash = iconService.sendTransaction(signedTransaction).execute();

// Print Transaction Hash
System.out.println("txHash:"+txHash);

// Output
txHash:0x6b17886de346655d96373f2e0de494cb8d7f36ce9086cb15a57d3dcf24523c8f
```

#### Check the Result

After sending the transaction, you can check the result with the returned hash value.

In this example, you can check your transaction result in every 2 seconds because of the block confirmation time.

If the transaction succeeds, you can check the `scoreAddress` from the result.

You can use the SCORE after the SCORE passes the audit and is finally accepted to deploy.

```java
// Checking the results with transaction hash
TransactionResult result = iconService.getTransactionResult(hash).execute();
System.out.println("transaction status(1:success, 0:failure):"+result.getStatus());
System.out.println("created score address:"+result.getScoreAddress());
System.out.println("waiting accept score...");

// Output
transaction status(1:success, 0:failure):1
created score address:cxd7fce67cc95b731dfbfdd8c8b34e8a5d0664a9ed
waiting accept score...
```

*For the 'TransactionResult', please refer to the IcxTransactionExample.*

#### Token transfer transaction

You can get the token SCORE address by checking the `scoreAddress` from the deploy transaction result above, and use this to send the token.

You can generate a KeyWallet using `CommonData.PRIVATE_KEY_STRING` just like in the case of `IcxTransactionExample`, then send 1 Token to `CommonData.ADDRESS_1`.

```java
Wallet wallet = KeyWallet.load(new Bytes(CommonData.PRIVATE_KEY_STRING));
Address toAddress = new Address(CommonData.ADDRESS_1);

// Deploy a token SCORE and get the SCORE address
Address tokenAddress = new DeployTokenExample().deploy(wallet);

int tokenDecimals = 18;	// token decimal
// 1 ICX -> 1000000000000000000 conversion
BigInteger value = IconAmount.of("1", tokenDecimals).toLoop();
```

Generate a transaction with the given parameters above.
You have to add receiving address and value to `RpcObject` to send token.

```java
// Network ID ("1" for Mainnet, "2" for Testnet, etc)
BigInteger networkId = new BigInteger("3");
// Transaction creation time (timestamp is in the microsecond)
long timestamp = System.currentTimeMillis() * 1000L;
// 'transfer' as a methodName means to transfer token
// https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md
String methodName = "transfer";

// Enter receiving address and the token value.
// You must enter the given key name ("_to", "_value"). Otherwise, the transaction will be rejected.
RpcObject params = new RpcObject.Builder()
	.put("_to", new RpcValue(toAddress))
	.put("_value", new RpcValue(value))
	.build();

// Create a raw transaction to transfer token (without stepLimit)
Transaction transaction = TransactionBuilder.newBuilder()
	.nid(networkId)
	.from(wallet.getAddress())
	.to(tokenAddress)
	.timestamp(new BigInteger(Long.toString(timestamp)))
	.call(methodName)
	.params(params)
	.build();

// Get an estimated step value
BigInteger estimatedStep = iconService.estimateStep(transaction).execute();

// Create a signedTransaction with the sender's wallet and the estimatedStep
SignedTransaction signedTransaction = new SignedTransaction(transaction, wallet, estimatedStep);
```

Call `sendTransaction` from `IconService` to check the transaction hash.

```java
// Send transaction
Bytes txHash = iconService.sendTransaction(signedTransaction).execute();

// Print transaction hash
System.out.println("txHash:"+txHash);

// Output
txHash:0x6b17886de346655d96373f2e0de494cb8d7f36ce9086cb15a57d3dcf24523c8f
```

#### Check the Result

You can check the result with the returned hash value of your transaction.

In this example, you check your transaction result in every 2 seconds because the block confirmation time is around 2 seconds.
Checking the result is as follows:

```java
// Check the result with the transaction hash
TransactionResult result = iconService.getTransactionResult(hash).execute();
System.out.println("transaction status(1:success, 0:failure):"+result.getStatus());

// Output
transaction status(1:success, 0:failure):1
```

*For the TransactionResult, please refer to the [ICX Transfer](#section-icx-transfer) example.*

#### Check the token balance

In this example, you can check the token balance before and after the transaction.

You can check the token balance by calling `balanceOf` from the token SCORE.

```java
KeyWallet wallet; /* create or load */
Address tokenAddress; /* returned from `new DeployTokenExample().deploy(wallet)` */
String methodName = "balanceOf"; /* Method name to check the balance */

// Enter the address to check balance.
// You must enter the given key name ("_owner"). Otherwise, your transaction will be rejected.
RpcObject params = new RpcObject.Builder()
    .put("_owner", new RpcValue(wallet.getAddress()))
    .build();

Call<RpcItem> call = new Call.Builder()
    .to(tokenAddress)
    .method(methodName)
    .params(params)
    .build();

RpcItem result = iconService.call(call).execute();
BigInteger balance = result.asInteger();
System.out.println("balance:"+balance));

// Output
balance:6000000000000000000
```

### Sync Block

This example shows how to read block information and print the transaction result for every block creation.

*Please refer to above for KeyWallet and IconService creation.*

#### Read block information

In this example, `getLastBlock` is called periodically in order to check the new blocks,

by updating the transaction information for every block creation.

```java
// Check the recent blocks
Block block = iconService.getLastBlock().execute();
System.out.println("block height:"+block.getHeight());

// Output
block height:237845
```

If a new block has been created, get the transaction list.

```java
List<ConfirmedTransaction> txList = block.getTransactions();
System.out.println("transaction hash:" + transaction.getTxHash());
System.out.println("transaction:" + transaction);

// Output
transaction hash:0x0d2b71ec3045bfd39f90da844cb03c58490fe364c7715cc299db346c1153fe0f
transaction:ConfirmedTransaction...stepLimit=0xfa0...value=0x21e19e...version=0x3...
```

You can check the following information using the `ConfirmedTransaction`:

- version : json rpc server version
- to : Receiving address of the transaction
- value: The amount of ICX coins to transfer to the address. If omitted, the value is assumed to be 0
- timestamp: timestamp of the transmitting transaction (unit: microseconds)
- nid : network ID
- signature: digital signature data of the transaction
- txHash : transaction hash
- dataType: A value indicating the type of the data item (call, deploy, message)
- data: Various types of data are included according to dataType.

#### Transaction output

After reading the `TransactionResult`, merge with `ConfirmedTransaction` to send ICX or tokens. Transaction output is as follows:

```java
TransactionResult txResult = iconService.getTransactionResult(transaction.getTxHash()).execute();

// Send ICX
if ((transaction.getValue() != null) &&
    (transaction.getValue().compareTo(BigInteger.ZERO) > 0)) {

    System.out.println("[Icx] status:" + txResult.getStatus() +
                       ",from:" + transaction.getFrom() +
                       ",to:" + transaction.getTo() +
                       ",amount:" + transaction.getValue());
}

// Send token
if (transaction.getDataType() != null &&
    transaction.getDataType().equals("call")) {

    RpcObject data = transaction.getData().asObject();
    String methodName = data.getItem("method").asString();

    if (methodName != null && methodName.equals("transfer")) {
        RpcObject params = data.getItem("params").asObject(); // SCORE params
        BigInteger value = params.getItem("_value").asInteger(); // value
        Address toAddr = params.getItem("_to").asAddress(); // to address value

        String tokenName = getTokenName(transaction.getTo());
        String symbol = getTokenSymbol(transaction.getTo());
        String token = String.format("[%s Token(%s)]", tokenName, symbol);
        System.out.println(token+",tokenAddress:" + transaction.getTo() +
                           ",status:" + txResult.getStatus() +
                           ",from:" + transaction.getFrom() +
                           ",to:" + toAddr +
                           ",amount:" + value);
    }
}
```

#### Check the token name & symbol

You can check the token SCORE by calling the `name` and `symbol` functions.

```java
Address tokenAddress; /* returned from `new DeployTokenExample().deploy(wallet)` */

Call<RpcItem> call = new Call.Builder()
    .to(tokenAddress)
    .method("name")
    .build();

RpcItem result = iconService.call(call).execute();
String tokenName = result.asString();

Call<RpcItem> call = new Call.Builder()
    .to(tokenAddress)
    .method("symbol")
    .build();

result = iconService.call(call).execute();
String tokenSymbol = result.asString();

System.out.println("tokenName:"+tokenName);
System.out.println("tokenSymbol:"+tokenSymbol);

// Output
tokenName:StandardToken
tokenSymbol:ST
```

## References

- [API Reference](http://www.javadoc.io/doc/foundation.icon/icon-sdk)
- [ICON JSON-RPC API v3](icon-json-rpc-v3)
- [ICON Network](the-icon-network)


## Licenses

This project follows the Apache 2.0 License. Please refer to [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) for details.
