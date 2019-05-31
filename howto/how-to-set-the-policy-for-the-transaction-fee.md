---
title: "How to set the policy for the transaction fee"
excerpt: ""
---

## Overview

Users must pay the transaction fee to execute a smart contract (SCORE) on the ICON Network. But the transaction fee charged to the users would hinder attracting more users into the SCORE service. Thus we provide new features called "Fee Sharing" and "Virtual Step" to solve this well-known problem.

This document shows how to use the Fee Sharing and Virtual Step, and what benefits service operators can get from the new features.

## Intended Audience

Service operators who want their users to use their services without any transaction fees.

## Purpose

Service operators are able to set some policy options for the transaction fee after following this document.

* Service operators can pay the transaction fee on behalf of the service users.
* Service operators can offset the fee burden by using the Virtual Step system.

## Prerequisites

* Basic concept of the transaction fee and Step on ICON Network
* Knowledge of the ICON JSON-RPC APIs.

## Make a SCORE which supports the Fee Sharing

Be default, all the transaction fee to execute a SCORE will be charged to the transaction sender. However, the SCORE can determine who will pay the transaction fee during its execution. To do this, we provide the following APIs that can be invoked on the SCORE method, which are declared in `IconScoreBase`.

* `get_fee_sharing_proportion(self) -> int`
    * Returns the current fee sharing proportion of the SCORE.
    * Return value
        * the current fee sharing proportion that the SCORE will pay
        * 100 means the SCORE will pay 100% of transaction fee on behalf of the transaction sender.
        * `int` value between 0 to 100
* `set_fee_sharing_proportion(self, proportion: int)`
    * Sets the proportion of the transaction fee that the SCORE will pay
    * If this method is invoked multiple times, the last proportion value will be used.

### SCORE Example

The following example sets the fee sharing proportion according to the white list that can be set separately. If the user is in the white list, some or all of the transaction fee will be shared by the SCORE operator as much as the proportion value set in the white list.

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
        self.ValueSet(self.tx.origin, proportion)
        self.set_fee_sharing_proportion(proportion)
```

### Transaction Result

0%
```json
{
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0xcfadd0ff7fc152f23c0ce92be4675d36743db11c25378a5a17560d286f641362",
        "blockHeight": "0x11",
        "blockHash": "0x8ae111527f885a0ca79e431c47624e35a01e2eb8169e406725ea785a9bf2cfc8",
        "txIndex": "0x0",
        "to": "cx216e1468b780ac1b54c328d19ea23a35a6899e55",
        "stepUsed": "0x20be8",
        "stepPrice": "0x2540be400",
        "cumulativeStepUsed": "0x20be8",
        "eventLogs": [],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

50%
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
        "logsBloom": "0x00000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000020002000000000000000100000000000000000080000004000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1",
        "stepUsedDetails": {
            "hx93a1562d85982de882b5fa4df05d42d35a7db0b1": "0x10de2",
            "cx216e1468b780ac1b54c328d19ea23a35a6899e55": "0x10de2"
        }
    },
    "id": 1
}
```

100%
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
        "logsBloom": "0x00000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000020002000000000000000000000000000000000080000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000000000000100000000000000002000080000000000000000000000000000000000000000020000000000000000000000000000",
        "status": "0x1",
        "stepUsedDetails": {
            "cx216e1468b780ac1b54c328d19ea23a35a6899e55": "0x21bc4"
        }
    },
    "id": 1
}
```

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

### Add a deposit

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
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
    },
    "id": 1
}
```

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
        "logsBloom": "0x00000001000000000000000000000000000000000000000000000000000000000010000000008000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000004000000000000000000000400000000000000000000000000000040000002000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000200080000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

### Withdraw a deposit

* SCORE owner can withdraw a deposit using the following JSON-RPC API
* Caution: if a SCORE owner wants to withdraw the deposit which does not expire, some  amounts for used virtual steps are deducted from the deposit.

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
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
    },
    "id": 1
}
```

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
        "logsBloom": "0x00000001000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000001000000000000000000000400000000000000000000000000000040000002000000000000000000000000000000000000000000000000000000000000000000000000000010000000000000000080000000000000200000000000000000000000020000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

## Summary

1. 거래 수수료 공유 비율을 정할 수 있는 set_fee_sharing_proportion() 함수를 사용하여 거래 처리 시 소비되는 수수료를 부담하는 비율을 조절할 수 있다.
3. Add deposit을 통해 virtual step을 발급받을 수 있다.
4. getScoreStatus를 이용하여 대상 SCORE의 deposit 상태 조회가 가능하다.
5. Withdraw deposit을 통해 거래 수수료에 사용했던 예치금을 찾아올 수 있다.

## References

- [ICON JSON RPC](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md#debug_estimateStep)
- [T-Bears]()
- [Java SDK]()
- [JavaScript SDK]()
- [Python SDK]()
- [Swift SDK]()
