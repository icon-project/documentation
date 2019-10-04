---
title: "How to set the policy for the transaction fees"
excerpt: ""
---

## Overview

Transaction senders must pay the transaction fees to execute a smart contract on the blockchain. The result of this transaction fee system was that all dApp users were required to pay transaction fees to use the service. This has been a major hurdle in attracting more users into the dApp service. With the ICON Transaction Fee 2.0 (Fee Sharing and Virtual Step), dApp service operators can choose to pay the transaction fees on behalf of the service users.

This document shows how to use the Fee Sharing and Virtual Step, and what benefits the service operators can get from the new features.

## Intended Audience

ICON dApp service operators who want the users to use their services without paying any transaction fees.

## Purpose

Service operators will be able to set the policy for the transaction fees after following this document.

* Service operators can pay the transaction fees on behalf of the service users.
* Service operators can offset the fee burden by using the Virtual Step system.

## Prerequisites

* Basic concept of the [transaction fees and Step on ICON Network](https://www.icondev.io/docs/transaction-fees)
* Knowledge of the [ICON JSON-RPC APIs](https://www.icondev.io/docs/icon-json-rpc-v3)
* [Writing SCORE](https://www.icondev.io/docs/writing-score)

## Make your SCORE support Fee Sharing

By default, all the transaction fees to execute a SCORE will be charged to the transaction sender. With ICON Fee2.0, the SCORE can determine who will pay the transaction fees during its execution. To do this, `IconScoreBase` provides the following APIs that can be invoked inside the SCORE's external methods that cause a state transition.

* `get_fee_sharing_proportion(self) -> int`
    * Returns the current fee sharing proportion of the SCORE.
    * Return value
        * the current fee sharing proportion that the SCORE will pay
        * 100 means the SCORE will pay 100% of transaction fees on behalf of the transaction sender.
        * `int` value between 0 to 100
* `set_fee_sharing_proportion(self, proportion: int)`
    * Sets the proportion of the transaction fees that the SCORE will pay
    * `proportion` should be between 0 to 100
    * If this method is invoked multiple times, the last proportion value will be used.

### SCORE Example
Below example SCORE pays the full or partial transaction fee that is required to invoke the `setValue` method for the registered users.
The following example sets the fee sharing proportion according to the whitelist that can be set separately. If the transaction sender (`self.tx.origin`) is in the whitelist, part of the transaction fees will be charged to the SCORE as per the proportion value set in the whitelist.

```python
from iconservice import *

class FeeSharing(IconScoreBase):

    @eventlog(indexed=1)
    def ValueSet(self, address: Address, proportion: int): pass

    def __init__(self, db: IconScoreDatabase):
        super().__init__(db)

        self._whitelist = DictDB("whitelist", db, int)
        self._value = VarDB("value", db, str)

    def on_install(self):
        super().on_install()

        self._value.set("No value")

    def on_update(self):
        super().on_update()

    def _check_owner(self):
        if self.tx.origin != self.owner:
            revert("Invalid SCORE owner")

    @staticmethod
    def _check_proportion(proportion: int):
        if not (0 <= proportion <= 100):
            revert(f"Invalid proportion: {proportion}")

    @external(readonly=True)
    def getProportion(self, address: Address) -> int:
        return self._whitelist[address]

    @external
    def addToWhitelist(self, address: Address, proportion: int = 100):
        self._check_owner()
        self._check_proportion(proportion)

        self._whitelist[address] = proportion

    @external(readonly=True)
    def getValue(self) -> str:
        return self._value.get()

    @external
    def setValue(self, value: str):
        self._value.set(value)

        proportion: int = self._whitelist[self.tx.origin]
        self.set_fee_sharing_proportion(proportion)
        self.ValueSet(self.tx.origin, proportion)
```

Note that `set_fee_sharing_proportion()` method should be called inside the SCORE method (`setValue` in the example above) that the SCORE is to pay the transaction fee for the transaction sender. Otherwise, the transaction sender will pay all of the transaction fees.

Also the transaction sender must have a minimum ICX balance to send the transaction, since it cannot be known beforehand whether the SCORE will pay the transaction fees without executing the transaction actually. However, the users balance will not be changed if the SCORE pays all of the transaction fees.

## Add deposit to the SCORE

After deploying the example SCORE above to the ICON Network, you need to deposit ICX to the SCORE for the Fee Sharing and this deposit action will generate Virtual Steps.
Currently, only the SCORE owner (who deployed the SCORE) can deposit ICX to the SCORE.
To add a deposit to the SCORE, use the following JSON-RPC API or the equivalent in ICON SDKs. See [Java SDK] and [Python SDK].

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1,
    "params": {
        "version": "0x3",
        "nid": "0x3",
        "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "value": "0x10f0cf064dd59200000",
        "stepLimit": "0x3000000",
        "timestamp": "0x58a1be6dac367",
        "nonce": "0x1",
        "signature": "Z+sc78SjGGsdch5kalcNqaK8+7ZX8M6SwaRYjrFopOoepLBok/sJ9EPulGxrDN4OodTqqYRA6KnuwGrNStomwAA=",
        "dataType": "deposit",
        "data": {
            "action": "add"
        }
    }
}
```

If the deposit `add` transaction was successful, you will see the transaction result like the following, that has `DepositAdded` eventlog.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x64b118d4a3c2b3b93362a0f3ea06e5519de42449523465265b85509041e69011",
        "blockHeight": "0x16",
        "blockHash": "0x2d082515c7a9098f6b1e88d42a3c11b227dc5e428aa28a97da7b6dcc22d0550c",
        "txIndex": "0x0",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "stepUsed": "0x1ba94",
        "stepPrice": "0x2540be400",
        "cumulativeStepUsed": "0x1ba94",
        "eventLogs": [
            {
                "scoreAddress": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
                "indexed": [
                    "DepositAdded(bytes,Address,int,int)",
                    "0x64b118d4a3c2b3b93362a0f3ea06e5519de42449523465265b85509041e69011",
                    "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb"
                ],
                "data": [
                    "0x10f0cf064dd59200000",
                    "0x13c680"
                ]
            }
        ],
        "logsBloom": "0x00000001000000000000000000000...0000000000000000010000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

The declaration of `DepositAdded` eventlog is as follows.

```python
@eventlog(indexed=2)
def DepositAdded(self, id: bytes, from_: Address, amount: int, term: int):
    pass
```
`id` is the deposit id (i.e., the transaction hash), `from_` is the transaction sender, `amount` is the amount of deposited ICX, and `term` is the deposit period in blocks, which is currently fixed to 1 month (1,296,000 blocks in 30 days).


## Invoke SCORE method and check the fee deduction result

Now you have a SCORE that has the ICX deposit.
Before sending a transaction, you need to add the user to the whitelist.
If a user who is in the whitelist calls the SCORE method (`setValue` in this case), part of the transaction fee will be charged to the SCORE according to the proportion set in the whitelist.
Here's an example of the transaction result, when the proportion is 100.
Note that a new `stepUsedDetails` field was added to show the list of accounts that pay the fees.
In this case, the SCORE pays all of the transaction fees.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x768a27f1591e0fe207b18a61632f50a9e32fd95898e3ef0e308d5c202ed5c48d",
        "blockHeight": "0x18",
        "blockHash": "0x3f94e2b1ba00af07bc4b2f8f6dfbe0cbaed5c608cd213d6e12b264e3db210bcd",
        "txIndex": "0x0",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "stepUsed": "0x21bc4",
        "stepPrice": "0x2540be400",
        "cumulativeStepUsed": "0x21bc4",
        "eventLogs": [
            {
                "scoreAddress": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
                "indexed": [
                    "ValueSet(Address,int)",
                    "hx6a94ebb8726b6f87aec5f7f1049bfcb1833f1bf4"
                ],
                "data": [
                    "0x64"
                ]
            }
        ],
        "logsBloom": "0x00000001000000000000000000000...0000000000000000010000000000000",
        "status": "0x1",
        "stepUsedDetails": {
            "cx216e1468b780ac1b54c328d19ea23a35a6899e55": "0x21bc4"
        }
    },
    "id": 1
}
```

If the proportion is 50, the user and the SCORE pay the transaction fees half and half.
Check the `stepUsedDetails` field in the following example how they are represented in this case.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x5ca11a63e24cdb837026d4a20165d0246ecfc7bc40d84eaa33bdf43538b318c8",
        "blockHeight": "0x17",
        "blockHash": "0x53e6c94078e6b94135fe773675493dcc47ea62e8878e48eacd3044f5068d7bf1",
        "txIndex": "0x1",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "stepUsed": "0x21bc4",
        "stepPrice": "0x2540be400",
        "cumulativeStepUsed": "0x43788",
        "eventLogs": [
            {
                "scoreAddress": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
                "indexed": [
                    "ValueSet(Address,int)",
                    "hx93a1562d85982de882b5fa4df05d42d35a7db0b1"
                ],
                "data": [
                    "0x32"
                ]
            }
        ],
        "logsBloom": "0x00000001000000000000000000000...0000000000000000010000000000000",
        "status": "0x1",
        "stepUsedDetails": {
            "hx93a1562d85982de882b5fa4df05d42d35a7db0b1": "0x10de2",
            "cx216e1468b780ac1b54c328d19ea23a35a6899e55": "0x10de2"
        }
    },
    "id": 1
}
```

Note that the transaction result will be the same as when there was no Fee 2.0 feature if either the user is not in the whitelist or the SCORE does not set the fee sharing proportion (i.e., there will be no `stepUsedDetails` in the transaction result).

## Check the deposit status of the SCORE

Service operators often need to check the deposit status of the SCORE to know how much deposit (or Virtual Steps) were consumed and when they need to deposit additional ICX to the SCORE to continue their services seamlessly.
To do this, we retrofit the existing Governance API, [`getScoreStatus`](https://github.com/icon-project/governance#getscorestatus), to show the deposit status of the SCORE.
Here's an example result of the API call.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "current": {
            "status": "active",
            "deployTxHash": "0x19793f41b8e64fc31190c6a70a103103da1f4bc81bc829fa72c852a5e388fe8c"
        },
        "depositInfo": {
            "scoreAddress": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
            "deposits": [
                {
                    "id": "0x64b118d4a3c2b3b93362a0f3ea06e5519de42449523465265b85509041e69011",
                    "sender": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
                    "depositAmount": "0x10f0cf064dd59200000",
                    "depositUsed": "0x0",
                    "created": "0x16",
                    "expires": "0x13c696",
                    "virtualStepIssued": "0x9502f9000",
                    "virtualStepUsed": "0x329a6"
                }
            ],
            "availableVirtualStep": "0x9502c665a",
            "availableDeposit": "0xf3f20b8dfa69d00000"
        }
    },
    "id": 1
}
```

## Withdraw the deposit

SCORE owners can withdraw the deposit by using the following JSON-RPC API.

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1,
    "params": {
        "version": "0x3",
        "nid": "0x3",
        "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "value": "0x0",
        "stepLimit": "0x3000000",
        "timestamp": "0x58a2bc315916c",
        "nonce": "0x1",
        "signature": "yYA8OsVB3RM6h8RdNdbS5/7D5pDWxJTIVIWX3a0O0TIbhpYF/36oQYQfx1iq1DGChZ29tfVxx8mY/x+BXSH6pAA=",
        "dataType": "deposit",
        "data": {
            "action": "withdraw",
            "id": "0x64b118d4a3c2b3b93362a0f3ea06e5519de42449523465265b85509041e69011"
        }
    }
}
```

Note that if the SCORE owner wants to withdraw the deposit which has not been expired, the same amount of ICX will be deducted from the deposit by the amount of Virtual Steps that were used.

Here's an example of the transaction result, when the SCORE owner withdraws the deposit.
If the deposit `withdraw` transaction was successful, you will see the transaction result like the following, that has `DepositWithdrawn` eventlog.


```json
{
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0xcd43ffc50418e025d06cbeb7d43cc6fad7c173f966b6764cb825b62459cc7e67",
        "blockHeight": "0x1b",
        "blockHash": "0xfb8617818c296891de7713c27134069c71a43ab1bdb71ee3cfc615040a9504bb",
        "txIndex": "0x0",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "stepUsed": "0x1fb6c",
        "stepPrice": "0x2540be400",
        "cumulativeStepUsed": "0x1fb6c",
        "eventLogs": [
            {
                "scoreAddress": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
                "indexed": [
                    "DepositWithdrawn(bytes,Address,int,int)",
                    "0x64b118d4a3c2b3b93362a0f3ea06e5519de42449523465265b85509041e69011",
                    "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb"
                ],
                "data": [
                    "0x10f0ce907c145e62800",
                    "0x75d1c1339d800"
                ]
            }
        ],
        "logsBloom": "0x00000001000000000000000000000...0000000000000000010000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

The declaration of `DepositWithdrawn` eventlog is as follows.

```python
@eventlog(indexed=2)
def DepositWithdrawn(self, id: bytes, from_: Address, returnAmount: int, penalty: int):
    pass
```
`id` is the deposit id (i.e., the transaction hash), `from_` is the transaction sender, `returnAmount` is the returned amount of deposited ICX, and `penalty` is the deducted ICX as the penalty if this is an early withdrawal.


## Summary

* Service operators will be able to pay transaction fees on behalf of the service users by calling `set_fee_sharing_proportion()` API from the SCORE method.
* Depositing ICX to the SCORE will generate Virtual Steps that can be used to pay transaction fees.
* `getScoreStatus` API of Governance can be used to check the current deposit status of the SCORE.
* Service operators can withdraw the deposit in the SCORE at any time, however, some penalty will be incurred if it is an early withdrawal.

## References

- [ICON JSON RPC]
- [T-Bears]
- [Java SDK]
- [Python SDK]

[ICON JSON RPC]: https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md
[T-Bears]: https://github.com/icon-project/t-bears
[Java SDK]: https://github.com/icon-project/icon-sdk-java
[Python SDK]: https://github.com/icon-project/icon-sdk-python
