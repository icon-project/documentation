---
title: "How to generate a transaction signature"
---

When a user sends a transaction, they need to sign the data with their own private key. There are two major steps involved in signing the transaction, serializing the original transaction data and generating a signature with user's own private key. This document describes the process of generating a digital signature of transaction data.

## Intended Audience

- Someone who wants to develop the program which communicates with ICON network with languages whose SDK is unsupported

## Purpose

Generating a valid transaction signature in ICON network.

## Prerequisite

- [Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
- [Transaction](https://github.com/icon-project/documentation/blob/develop/icon-key-concepts/transactions.md)
- [ICON JSON-RPC API v3](https://github.com/icon-project/documentation/blob/develop/references/json-rpc/icon-json-rpc-v3.md)

## How to Serialize Transaction Data

Before signing the data, the data needs to be serialized as bytes.
This section describes how to serialize the transaction data.
This document does not describe how to make the transaction data itself. Transaction message specification is defined in [JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md).

### Precondition

Transaction data is in JSON format with some restrictions.

#### Allowed types in JSON

| Type       | Description                                                  |
| ---------- | ------------------------------------------------------------ |
| String     | Normal string without U+0000 (NULL) character. ex) “Value”   |
| Dictionary | Pairs of key and value. ex) {“key1”: “value1”, “key1”: “value2”} |
| Array      | Series of values. ex) [“value1”, “value2”]                   |
| Null       | null                                                         |

### Serialize

ICON follows the JSON-RPC v2.0 protocol spec. A signed transaction request looks like the following: 

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "hx5bfdb090f43a808005ffc27c25b213145e80b7cd",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x1",
        "nonce": "0x1",
        "signature": "X1tpJdHBvqroonpTbdsNEur7KAeYcZd9XGa39AkW51Uck8EqgJnioedm5W2jZSQuBzZJHWm0Uf5BeXSmXoOByAA="
    }
}
```

Transaction data is serialized by concatenating key, value pairs in `params` with `.` as a delimiter. Our final goal is generating the `signature` of transaction data, therefore, the `signature` field shown above example is not part of the data to be serialized. Adding the method name, “icx_sendTransaction”, to the serialized string as a prefix completes the serialization process.
```
icx_sendTransaction.<key1>.<value1>....<keyN>.<valueN>
```

Make sure that all the special characters in string must be UTF-8 encoded.

#### String type

Apply UTF-8 encoding to the text. Characters listed below should be escaped with `\`. String must not have any null character, U+0000.

| Name  | Backslash (REVERSE SOLIDUS) | Period (FULL STOP) | Left curly bracket | Right curly bracket | Left square bracket | Right square bracket |
| ----- | --------------------------- | ------------------ | ------------------ | ------------------- | ------------------- | -------------------- |
| Shape | *\\*                        | **.**              | **{**              | **}**               | **[**               | **]**                |
| UTF-8 | 0x5C                        | 0x2E               | 0x7B               | 0x7D                | 0x5B                | 0x5D                 |

#### Dictionary type

Enclosed with `{` and `}`, and key/value pairs are separated with `.`. Every keys in dictionary are string type, therefore, the same encoding rules apply. The order of keys in the serialized data follows the natural ordering of the UTF-8 encoded byte comparison (it’s same as Unicode string order).

```
{<key1>.<value1>.<key2>.<value2>....<keyN>.<valueN>}
```

Example:

```json
{
    "method": "transfer",
    "params": {
        "to": "hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b",
        "value": "0x1"
    }
}
```

Serialized as

``` 
{method.transfer.params.{to.hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b.value.0x1}}
```

#### Array type

Enclosed with `[` and `]`. All values are separated with `.`.

``` 
[<value1>.<value2>.<value3>....<valueN>]
```

#### Null type

Null will be represented as an escaped zero.

```
\0
```

### Example

#### ICX transfer

Original JSON request:

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "hx5bfdb090f43a808005ffc27c25b213145e80b7cd",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x1",
        "nonce": "0x1"
    }
}
```

Serialized params:

```
icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.nid.0x1.nonce.0x1.stepLimit.0x12345.timestamp.0x563a6cf330136.to.hx5bfdb090f43a808005ffc27c25b213145e80b7cd.value.0xde0b6b3a7640000.version.0x3
```

#### SCORE API invocation

Original JSON request:

```json

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
        "nid": "0x1",
        "nonce": "0x1",
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

Serialized params:

 ```
icx_sendTransaction.data.{method.transfer.params.{to.hxab2d8215eab14bc6bdd8bfb2c8151257032ecd8b.value.0x1}}.dataType.call.from.hxbe258ceb872e08851f1f59694dac2558708ece11.nid.0x1.nonce.0x1.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.version.0x3
 ```

## How to Create Transaction Signature

This section describes how to create a valid signature for a transaction using the private key associated with a specific wallet address.

### Required data

#### Private key

The private key of the `from` address from which the transaction request was made.

#### Transaction hash 

32 bytes (256-bit) hash data which is created by hashing the serialized transaction data using SHA3_256.

### Create signature

The first step is making a serialized signature. Using the `secp256k1` library, create a recoverable ECDSA signature of a transaction hash. This ensures that the transaction was originated from the private key owner, and the transaction message received by the recipient is not compromised. The resulting output should be 64 bytes serialized signature (R, S) with 1 byte recovery id (V).

The final step is to encode the generated signature as a Base64-encoded string.

## Steps of How-to

### How to Generate Transaction Signature in Python

Below is the example of creating a signature in python. 

Here is a sample transaction request message. We will create a signature for the transaction data.

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x1"
    }
}
```

- step1: Serialize transaction data.

```
icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.nid.0x1.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.value.0xde0b6b3a7640000.version.0x3
```

- step2: Create a transaction signature.
  - step2-1: Hash the serialized transaction data.
  - step2-2: Create a recoverable ECDSA signature.
  - step2-3: Encode the generated recoverable ECDSA signature as a Base64-encoded string.

```python
import base64
import hashlib
import secp256k1

serialized_transaction = "icx_sendTransaction.from.hxbe258ceb872e08851f1f59694dac2558708ece11.nid.0x1.stepLimit.0x12345.timestamp.0x563a6cf330136.to.cxb0776ee37f5b45bfaea8cff1d8232fbb6122ec32.value.0xde0b6b3a7640000.version.0x3"

# transaction_hash
# result: b'z\xdc\xa3\xc5@\x19{\xc0\xc5\xe3b\xc3I\x84&k\xeb\xbc\xd2\xda\xe2\xfd\x06\x08\x95TR[\x9b\xfc\xd0\xff'
msg_hash = hashlib.sha3_256(serialized_transaction.encode()).digest()

# prepare the private key (this private key is for example purpose)
private_key = b'\x870\x91*\xef\xedB\xac\x05\x8f\xd3\xf6\xfdvu8\x11\x04\xd49\xb3\xe1\x1f\x17\x1fTR\xd4\xf9\x19mL'

# create a private key object
private_key_object = secp256k1.PrivateKey(private_key)

# create a recoverable ECDSA signature
recoverable_signature = private_key_object.ecdsa_sign_recoverable(msg_hash, raw=True)

# convert the result from ecdsa_sign_recoverable to a tuple composed of 64 bytes and an integer denominated as recovery id.
signature, recovery_id = private_key_object.ecdsa_recoverable_serialize(recoverable_signature)
recoverable_sig = bytes(bytearray(signature) + recovery_id.to_bytes(1, 'big'))

# base64 encode
transaction_signature = base64.b64encode(recoverable_sig)

# transaction_signature: b'HNsFOK1qRkVKMB8ePZhKg/ELmT53MmnZn4ftt2sD69VdobB94BT0h52Bb8ven53186A9u+eIiIiWrSu8VjMUpwE='
```

## Summary

So far, we have learned about the rule of serializing transaction data, a process of converting from serialized transaction data to transaction signature and the list of libraries used to generate a signature. Be aware that you can not generate a valid transaction signature if you omit process or break any rule.

## Tips or FAQs

## References
