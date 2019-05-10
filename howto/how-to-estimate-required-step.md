---
title: "How to Estimate Required STEP"
excerpt: ""
---

To make the submitted TX be executed successfully, the `step_limit` property should be input properly. 
This document explains how to estimate the required STEPs.


## Intended Audience

Anybody who sends transactions to ICON Network using their own systems(using SDKs, TBears or JSON-RPC for ICON).

## Purpose

Users are able to estimate the required STEPs for their transactions following this document.

## Prerequisite 

- Knowledge of the ICON JSON RPC.
- Need to know how to submit a transaction to the ICON Network.

## How-To 

The STEP estimation can be requested through JSON-RPC, similar form to sending transactions.

#### Sending Transaction

Transactions will be submitted as the following example.

This example shows the transaction request that invoking `transfer` on an IRC2 contract.

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxcced4d83a03098f5976b5ed339b0cab3e51ca1f9",
        "to": "cx65fc79aa09e867fd51d0c8361f42cb38bc1f08f3",
        "stepLimit": "0x30d40",
        "timestamp": "0x583f58f62eae0",
        "nid": "0x3",
        "nonce": "0x1",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "_to": "hx25cd4028f055457b8fbdae1dbd8b5fe465248e55",
                "_value": "0x8ac7230489e80000"
          }
        },
        "signature": "rBaH0U6y85y1CWp/JbalUbzLVGjtGYG+hut/G5o30vBxhWoxPYtSYBQu6X0Tak1SdcnlZSCJL7DeOeKmI4y+5wE="
    }
}
```

#### Estimate

The estimation request is quite similar to the above except the `stepLimit`, `signature` of the `params` fields.

The `stepLimit`, `signature` fields are unnecessary because it should not be limited STEPs when STEP calculating, and it is not necessary to verify the sender.

The following example shows the STEP estimation request of the transaction above.

```json
{
    "jsonrpc": "2.0",
    "method": "debug_estimateStep",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxcced4d83a03098f5976b5ed339b0cab3e51ca1f9",
        "to": "cx65fc79aa09e867fd51d0c8361f42cb38bc1f08f3",
        "timestamp": "0x583f58f62eae0",
        "nid": "0x3",
        "nonce": "0x1",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "_to": "hx25cd4028f055457b8fbdae1dbd8b5fe465248e55",
                "_value": "0x8ac7230489e80000"
            }
        }
    }
}
```

#### Response

If there are no exceptions, the response shows the estimated STEPs as follow.

```json
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": "0x23f8c"
}
```

#### Failure

If there exist exceptions, the response shows the exception message.

```json
{
    "jsonrpc": "2.0",
    "id": 1234,
    "error": {
        "code": -32602,
        "message": "JSON schema validation error: 'version' is a required property"
    }
}
```

## Using T-Bears (NOT CONFIRMED)

Using T-Bears, the estimation can be queried by `estimate` command.

```shell
(venv)$ tbears estimate transfer.json
estimated steps: 147340
```

For more details [here]()

## Using SDKs (NOT CONFIRMED)

Official SDKs support the estimation functionality.

### Java (NOT CONFIRMED)

```java
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(wallet.getAddress())
    .to(scoreAddress)
    .nonce(new BigInteger("1000000"))
    .call("transfer")
    .params(params)
    .build();

Request<BigInteger> request = iconService.estimateStep(transaction);

try {
    BigInteger estimatedSteps = request.execute();
    ...
} catch (...) {
    ...
}        
```

For more details [here]()

### JavaScript (NOT CONFIRMED)

```javascript
const txObj = new CallTransactionBuilder()
    .from('hx902···3b5')
    .to('cx350···48d')
    .stepLimit(IconConverter.toBigNumber('2000000'))
    .nid(IconConverter.toBigNumber('3'))
    .nonce(IconConverter.toBigNumber('1'))
    .version(IconConverter.toBigNumber('3'))
    .timestamp((new Date()).getTime() * 1000)
    .method('transfer')
    .params({
        _to: 'hxd00···497',
        _value: IconConverter.toHex(IconAmount.of(1, IconAmount.Unit.ICX).toLoop()),
    })
    .build()

const estimatedSteps = await iconService.estimateStep(txObj).execute();
```

For more details [here]()

### python (NOT CONFIRMED)

```python
transaction = CallTransactionBuilder()\
    .from_(wallet.get_address())\
    .to("cx00...02")\
    .step_limit(1000000)\
    .nid(3)\
    .nonce(100)\
    .method("transfer")\
    .params(params)\
    .build()

estimated_steps = icon_service.estimate_step(transaction)
```

For more details [here]()

### swift (NOT CONFIRMED)

```swift
let call = CallTransaction()
    .from(wallet.address)
    .to(scoreAddress)
    .stepLimit(BigUInt(1000000))
    .nid(self.iconService.nid)
    .nonce("0x1")
    .method("transfer")
    .params(["_to": to, "_value": "0x1234"])

let request: Request<BigUInt> = iconService.estimateStep(call)
let result = request.execute()
```

For more details [here]()

## Summary

With the `debug_estimateStep` API, users can know how much STEPs are required for a particular transaction at the current block state.

However, the queried value is just an **ESTIMATED** value. 
The required STEPs are variable by the current block state, the block state may not be the same between when estimating and submitting.


## References

- [ICON JSON RPC](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md#debug_estimateStep)
- [T-Bears]()
- [Java SDK]()
- [JavaScript SDK]()
- [Python SDK]()
- [Swift SDK]()
