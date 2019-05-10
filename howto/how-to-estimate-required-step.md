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

STEP estimation can be requested through JSON-RPC, similar form to sending transactions.

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
        "from": "hx0000000000000000000000000000000000000001",
        "to": "cx0000000000000000000000000000000000000001",
        "stepLimit": "0x123456",
        "timestamp": "0x563a6cf330136",
        "nid": "0x3",
        "nonce": "0x1",
        "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA=",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "to": "hx0000000000000000000000000000000000000002",
                "value": "0x8ac7230489e80000"
            }
        }
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
        "from": "hx0000000000000000000000000000000000000001",
        "to": "cx0000000000000000000000000000000000000001",
        "timestamp": "0x563a6cf330136",
        "nid": "0x3",
        "nonce": "0x1",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "to": "hx0000000000000000000000000000000000000002",
                "value": "0x8ac7230489e80000"
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

<!--
## Using T-Bears


## Using SDKs

### Java

### JavaScript

### python

-->


## Summary

With the `debug_estimateStep` API, users can know how much STEPs are required for a particular transaction at the current block state.

However, the queried value is just an **ESTIMATED** value. 
The required STEPs are variable by the current block state, it may not be the same state between at estimating TX and at submitting TX.


## References

- [ICON JSON RPC](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md#debug_estimateStep)
<!--
- [T-Bears]()
- [Java SDK]()
- [JavaScript SDK]()
- [Python SDK]()
-->
