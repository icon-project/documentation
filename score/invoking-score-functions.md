---
title: "Invoking SCORE functions"
---

Describe How to invoke SCORE functions.

## Intended Audience

* Mid  
* Experienced

## Purpose 

Understand how to invoke SCORE functions.

## Prerequisite 

* [score-overview](https://github.com/icon-project/documentation/blob/develop/score/score-overview.md)

## Calling Read-Only method

- Does not make state transition. Thus, Don't need to sign.
- Write json to the specified format, and send a post request.
- You can send request by using curl or SDKs ([java](https://github.com/icon-project/documentation/blob/develop/references/java-sdk/java-sdk-reference.md), [python](https://github.com/icon-project/documentation/blob/develop/references/python-sdk/quickstart.md), [javascript](https://github.com/icon-project/documentation/blob/develop/references/javascript-sdk/quickstart.md) ..)

### Json Format

| KEY         | VALUE type                    | Description                                    |
| :---------- | :---------------------------- | :--------------------------------------------- |
| from        | T_ADDR_EOA     | Message sender's address.                      |
| to          | T_ADDR_SCORE | SCORE address that will handle the message.    |
| dataType    | T_DATA_TYPE   | `call` is the only possible data type.         |
| data        | T_DICT                        | See [Parameters - data](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#sendtxparameterdata). |
| data.method | String                        | Name of the function.                          |
| data.params | T_DICT                        | Parameters to be passed to the function.       |

### Returns

- Values returned by the executed SCORE function.

### Example

```javascript
// Request
{
    "jsonrpc": "2.0",
    "method": "icx_call",
    "id": 1234,
    "params": {
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
        "dataType": "call",
        "data": {
            "method": "get_balance",
            "params": {
                "address": "hx1f9a3310f60a03934b917509c86442db703cbd52"
            }
        }
    }
}

// Response - success
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": "0x2961fff8ca4a62327800000"
}

// Response - error
{
    "jsonrpc": "2.0",
    "id": 1234,
    "error": {
        "code": -32602,
        "message": "Invalid params"
    }
}
```

## Invoking Writable method

- Writable methods cause state transition. So, need to sign.
- Write json to the specified format, and send a post request.
- It is very difficult to call the writable function without using sdk because you need to sign it. Use SDK you are familiar with. ([java](https://github.com/icon-project/documentation/blob/develop/references/java-sdk/java-sdk-reference.md), [python](https://github.com/icon-project/documentation/blob/develop/references/python-sdk/quickstart.md), [javascript](https://github.com/icon-project/documentation/blob/develop/references/javascript-sdk/quickstart.md) ..)

### Json format

| KEY       | VALUE type                                                 | Required | Description                                                  |
| :-------- | :--------------------------------------------------------- | :------: | :----------------------------------------------------------- |
| version   | [T_INT](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_INT)                                            | required | Protocol version (`0x3` for V3)                              |
| from      | [T_ADDR_EOA](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_ADDR_EOA)                                  | required | EOA address that created the transaction                     |
| to        | [T_ADDR_EOA](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_ADDR_EOA) or [T_ADDR_SCORE](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_ADDR_SCORE) | required | EOA address to receive coins, or SCORE address to execute the transaction. |
| value     | [T_INT](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_INT)                                            | optional | Amount of ICX coins in loop to transfer. When omitted, assumes 0. (1 icx = 1 ^ 18 loop) |
| stepLimit | [T_INT](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_INT)                                            | required | Maximum step allowance that can be used by the transaction.  |
| timestamp | [T_INT](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_INT)                                            | required | Transaction creation time. timestamp is in microsecond.      |
| nid       | [T_INT](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_INT)                                            | required | Network ID (`0x1` for Mainnet, `0x2` for Testnet, etc)       |
| nonce     | [T_INT](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_INT)                                            | optional | An arbitrary number used to prevent transaction hash collision. |
| signature | [T_SIG](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_SIG)                                            | required | Signature of the transaction.                                |
| dataType  | [T_DATA_TYPE](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#T_DATA_TYPE)                                | optional | Type of data. (call, deploy, or message)                     |
| data      | T_DICT or String                                           | optional | The content of data varies depending on the dataType. See [Parameters - data](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md#sendtxparameterdata). |
| data.method | String | required | Name of the functions to invoke in SCORE |
| data.params | T-DICT | optional | Function parameters |


### Returns

- Transaction hash ([T_HASH](#T_HASH)) on success
- Error code and message on failure

### Example
- **SCORE function call**

```javascript
// Request
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x3",
        "nonce": "0x1",
        "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA=",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "to": "hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b",
                "value": "0x1"
            }
        }
    }
}
```

- **Responses**
```javascript
// Response - success
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": "0x4bf74e6aeeb43bde5dc8d5b62537a33ac8eb7605ebbdb51b015c1881b45b3aed" // transaction hash
}

// Response - error
{
    "jsonrpc": "2.0",
    "id": 1234,
    "error": {
        "code": -32600,
        "message": "Invalid signature"
    }
}

// Response - error
{
    "jsonrpc": "2.0",
    "id": 1234,
    "error": {
        "code": -32601,
        "message": "Method not found"
    }
}
```

## References

* [ICON JSON RPC] (https://github.com/icon-project/icon-rpc-server)
