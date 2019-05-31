---
title: "How to set the policy for the transaction fee"
excerpt: ""
---

## Overview

Users should have paid the transaction fee for executing a transaction on existing ICON Network. But this transaction fee was the large obstacle that service owners draw a lot of users to their services. Thus we provide the new features called "fee sharing" and "virtual step" to solve this well-known problem.

This document introduces how to use fee sharing and virtual step and what benefits service owners can get from the new features.

## Intended Audience

Service owners who want their users to use their dApp services without any transaction fees.

## Purpose

Service owners are able to set some policy options for the transaction fee following this document.

* Service owners can pay the transaction fee instead of service users.
* Service owners can save transaction fees consumed while running their services with the new solution **"virtual step"**.

## Prerequisite 

* Basic concept of transaction fee and step on ICON Network
* Knowledge of the ICON JSON-RPC APIs.

## How-To

### Make a SCORE which supports fee sharing

SCORE에서 TX를 처리하는데 필요한 수수료를 사용자 대신 SCORE가 부담하도록 하기 위해서는 먼저 SCORE가 부담하는 수수료 비율을 정해주어야 한다. 기본 동작은 사용자가 TX의 모든 수수료를 부담한다.

SCORE 내에서 수수료 부담 비율을 조절하기 위해서 다음의 IconScoreBase 메소드들을 제공한다.

* get_fee_sharing_proportion(self) -> int
	* 현재 설정되어 있는 SCORE의 수수료 부담 비율을 리턴한다.
	* Return value
		* The proportion of the transaction fee that SCORE will pay
		* 100 means that SCORE will pay 100% of transaction fee instead of transaction sender.
		* Integer between 0 to 100
* set_fee_sharing_proportion(self, proportion: int)
	* SCORE의 수수료 부담 비율을 재설정한다.
	* 여러 번 호출될 경우 마지막 값으로 수수료 부담 비율이 결정된다.

#### Example SCORE
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
        self.ValueSet(self.tx.origin, proportion)

        proportion: int = self._whitelist[self.tx.origin]
        self.set_fee_sharing_proportion(proportion)

```

#### Transaction Result

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
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxbe258ceb872e08851f1f59694dac2558708ece11",
        "value": "0x10f0cf064dd59200000",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x3",
        "nonce": "0x1",
        "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA=",
        "dataType": "deposit",
        "data": {
            "action": "add"
        }
    }
}
```

### Withdraw a deposit

* SCORE owner can withdraw a deposit using the following JSON-RPC API
* Caution: if SCORE owners want to withdraw the deposit which does not expires, they can do it with some penalty which is the same as the quantity of consumed virtual steps.

```json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxbe258ceb872e08851f1f59694dac2558708ece11",
        "to": "cxbe258ceb872e08851f1f59694dac2558708ece11",
        "stepLimit": "0x12345",
        "timestamp": "0x563a6cf330136",
        "nid": "0x3",
        "nonce": "0x1",
        "signature": "VAia7YZ2Ji6igKWzjR2YsGa2m53nKPrfK7uXYW78QLE+ATehAVZPC40szvAiA6NEU5gCYB4c4qaQzqDh2ugcHgA=",
        "dataType": "deposit",
        "data": {
            "action": "withdraw",
            "id": "0x4bf74e6aeeb43bde5dc8d5b62537a33ac8eb7605ebbdb51b015c1881b45b3111"
        }
    }
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
