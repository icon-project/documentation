---
title: "How to set the policy for the transaction fee"
excerpt: ""
---

기존 아이콘 망에서 사용자들은 거래를 처리하기 위해서 거래 수수료를 지불해야 한다.
Users should have paid the transaction fee for executing a transaction in existing ICON Network.
그러나 이러한 거래 수수료는 서비스 사용자를 끌어들이는데 치명적인 요소가 된다.
This document introduces the new policy for the transaction fee.

## Intended Audience

Service owners who want their users to use their dApp services without any transaction fees.

## Purpose

Service owners are able to set some policy options for the transaction fee following this document.
* Service owners can pay the transaction fee instead of service users.
* Service owners want to use the new feature called virtual step to save the transaction fee.

## Prerequisite 

* Knowledge of the ICON JSON-RPC.
* Need to know how to submit a transaction to the ICON Network.

## How-To

### Make a SCORE which supports fee sharing

### Add a deposit

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "icx_sendTransaction",
    "params": {
        "from": "hx4873b94352c8c1f3b2f09aaeccea31ce9e90bd31",
        "to": "cx4d6f646441a3f9c9b91019c9b98e3c342cceb114",
        "value": "0x10f0cf064dd59200000",
        "dataType": "deposit",
        "data": {
            "action": "add"
        }
    }
}
```

### Withdraw a deposit

SCORE owner can withdraw a deposit using the following JSON-RPC API

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "icx_sendTransaction",
    "params": {
        "from": "hx4873b94352c8c1f3b2f09aaeccea31ce9e90bd31",
        "to": "cx4d6f646441a3f9c9b91019c9b98e3c342cceb114",
        "value": "0x10f0cf064dd59200000",
        "dataType": "deposit",
        "data": {
            "action": "withdraw",
            "id": "0x78a7286833a17fc2b614da54e5a20d80748f041c81485244bf557445b19f5c07"
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