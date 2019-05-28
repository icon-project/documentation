---
title: "How to Estimate Required STEP"
excerpt: ""
---

It is important to set a proper `stepLimit` value in your transaction to make the submitted transaction executed successfully. 
This document explains how to systemically estimate the required Step.


## Intended Audience

Anyone who sends transactions to the ICON Network

## Purpose

To learn how to estimate the required Step for their transactions using ICON SDKs, T-Bears CLI or  JSON-RPC API calls.

## Prerequisite 

- Understand how ICON JSON-RPC APIs work.
- Need to know how to submit a transaction to the ICON Network using various tools.

## How-To

The mechanism of requesting Step estimation is very similar to sending transactions.

### Request and Response
#### Transaction Request

The following example is a transaction request message that will invoke the `transfer` method on the IRC2 token contract.

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

#### Step Estimate Request

The step estimation request is the same as the above example except the `stepLimit` and `signature`.
The `stepLimit` and `signature` fields are unnecessary because the step estimation process does not verify the sender, nor limit the execution until the cumulative step reaches the `stepLimit`. 

The following example is the Step estimation request for the above token transfer transaction. You can simply remove the `stepLimit` and `signature` fields from the original transaction request.

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

#### Response - Success

If there are no exceptions, the response will return the estimated Step usage as follows.

```json
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": "0x23f8c"
}
```

#### Response - Fail

If there is an exception, the response will show the error message.

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

### JSON-RPC API call
Debug API Endpoint : <scheme>://<host>/api/debug/v3

You can make a step estimation request using curl from the CLI as follows. Note that in the below command example, `stepEstimationRequest.json` is the file that contains the request message. 
```bash
$ curl -H "Content-Type: application/json" -d @stepEstimationRequest.json https://bicon.net.solidwallet.io/api/debug/v3
{"jsonrpc": "2.0", "result": "0x23f8c", "id": 1234}
```

### Java 

Coming Soon

### Python 

Coming Soon

### JavaScript 

TBA

### Swift

TBA

### T-Bears 

TBA

## Summary

With the `debug_estimateStep` JSON-RPC API, developers can estimate how much Step will be required for a particular transaction under the current block state.

However, please be aware that the returned value is just an **ESTIMATION**. 
The block state may not reamain the same between the Step estimation and the actual transaction execution time. The required Step can be different if the block state changes.


## References

- [ICON JSON RPC](icon-json-rpc-v3#section-debug_estimatestep)
- [T-Bears]()
- [Java SDK]()
- [JavaScript SDK]()
- [Python SDK]()
- [Swift SDK]()
