---
title: "Step Estimation"
excerpt: ""
---

As the ICON Yellowpaper describes, transaction fees are charged depending on used STEPs. 
It may be needed for users to know how much STEPs are required for certain transactions.
This document guides users the way to estimate it. 


## Intended Audience

Anybody who sends transactions to ICON Network using their own systems(using SDKs, TBears or JSON-RPC for ICON).


## Purpose

Users are able to estimate the required STEPs for their transactions following this document.


## Prerequisite

- Knowledge of the ICON JSON RPC.
- Need to know how to submit a transaction to the ICON Network.


## STEP

Fees incurred when executing a smart contract on the ICON Network depend on the amount of network resources used, and the unit 'STEP' is applied for measurement.

There are three major items that charge STEPs.
- Smart Contract Function Usage
- Blockchain Database Usage
- Amount of Transaction Data

In order for the submitted TX to be executed successfully, the value of the step_limit property should be enough to operate the TX, and the sender account should be eligible to pay as much as the input value of step_limit.


## STEP Estimation

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

Details of the sending transactions are [here](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md#icx_sendtransaction).

#### Estimation Request

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

If there are no exceptions, the response shows the estimated STEPs as follow.

```json
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": "0x23f8c"
}
```

## Using T-Bears (NOT CONFIRMED)

Using T-Bears, the estimation can be queried by `estimate` command.

```shell
(venv)$ tbears estimate transfer.json
estimated steps: 147340
```

For more details [here]()


## Summary

As the above tutorial, users can know how much STEPs are required for a particular transaction at the current block state using `debug_estimateStep` API.

However, the queried value is just an **ESTIMATED** value. 
The required STEPs are variable by the current block state, the block state may not be the same between when estimating and submitting.


## References

- [ICON Yellowpaper](https://icon.foundation/resources/file/ICON_Yellowpaper_Transactionfee_EN_V1.0.pdf)
- [ICON JSON RPC](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md#debug_estimateStep)
- [T-Bears]()
