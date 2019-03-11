---
title: "Writing SCORE"
---

## Overview

Described about How to write SCORE.  
This document offers "Syntax", "Limit", "API reference" and "Audit Checklist"

## Intended Audience

* Mid  
* Experienced

## Purpose 

You can understand how to write SCORE

## Prerequisite 

* [score-overview](https://github.com/icon-project/documentation/blob/master/score/score-overview.md)
* [score-by-example](https://github.com/icon-project/documentation/blob/master/score/score-by-example.md)
* [t-bears-tutorial](https://github.com/icon-project/t-bears)

## Package.json
   
Let's make SCORE project using T-Bears
```console
$ tbears init hello_world HelloWorld
```
After that, you can't miss json file in root path.
This json file describes like this.
```json
{
    "version": "0.0.1",
    "main_module": "hello_world",
    "main_score": "HelloWorld"
}
```

### if you want to point main_module as submodule?
```json
{
    "version": "0.0.1",
    "main_module": "sub_module.hello_world",
    "main_score": "HelloWorld"
}
```

## Syntax
### Type Hints

Type hinting is highly recommended for the input parameters and return value.  
When querying SCORE’s APIs, API specification is generated based on its type hints.  
If type hints are not given, only function names will return.

Possible data types for function parameters are int, str, bytes, bool, Address.  
List and Dict type parameters are not supported yet.  
Returning types can be int, str, bytes, bool, Address, List, Dict.

```python
@external(readonly=True)
def func1(arg1: int, arg2: str) -> int:
    return 100
```


### Exception handling

When you handle exceptions in your contract, it is recommended to use revert function rather 
than using an exception inherited from IconServiceBaseException.


### IconScoreBase (The highest parent class)

Every classes must inherit IconScoreBase.  
Contracts not derived from IconScoreBase can not be deployed.

#### _\_init_\_ 

This is a python init function.  
This function is called when the contract is loaded at each node.
Member variables can be declared here, however, Declaring member variables which not managed by states is prohibited.  
Utilities of DB such as VarDB, DictDB, ArrayDB can be declared here as a member variable as follows.

```python
def __init__(self, db: IconScoreDatabase) -> None:
    # Also, parent’s init function must be called as follows.
    super().__init__(db)

    self._total_supply = VarDB(self._TOTAL_SUPPLY, db, value_type=int)
    self._decimals = VarDB(self._DECIMALS, db, value_type=int)
    self._balances = DictDB(self._BALANCES, db, value_type=int)

```

#### on_install

This function is called when the contract is deployed for the first time, 
and will not be called again on contract update or deletion afterward.  
This is the place where you initialize the state DB.

#### VarDB, DictDB, ArrayDB  

VarDB, DictDB, ArrayDB are utility classes wrapping the state DB.  
A``key`` can be a number or characters, and ``value_type`` can be ``int``, ``str``, ``Address``, and ``bytes``.  
If the ``key`` does not exist, these classes return 0 when ``value_type`` is ``int``, return ""
when ``str``, return ``None`` when the ``value_type`` is ``Address`` or ``bytes``.  
VarDB can be used to store simple key-value state, and DictDB behaves more like python dict.  
DictDB does not maintain order, whereas ArrayDB, which supports length and iterator, maintains order.


**VarDB(‘key’, ‘target db’, ‘return type’)**

Example) Setting theloop for the key name on the state DB:

```python
VarDB('name', db, value_type=str).set('theloop')
Example) Getting value by the key name:

name = VarDB('name', db, value_type=str).get()
print(name) ## 'theloop'
```

**DictDB(‘key’, ‘target db’, ‘return type’, ‘dict depth (default is 1)’)**

Example 1) One-depth dict (test_dict1[‘key’]):

```python
test_dict1 = DictDB('test_dict1', db, value_type=int)
test_dict1['key'] = 1 ## set
print(test_dict1['key']) ## get 1

print(test_dict1['nonexistence_key']) # prints 0 (key does not exist and value_type=int)
```

Example 2) Two-depth dict (test_dict2[‘key1’][‘key2’]):

```python
test_dict2 = DictDB('test_dict2', db, value_type=str, depth=2)
test_dict2['key1']['key2'] = 'a' ## set
print(test_dict2['key1']['key2']) ## get 'a'

print(test_dict2['key1']['nonexistent_key']) # prints "" (key does not exist and value_type=str)
```

If the depth is more than 2, dict[key] returns new DictDB.  
Attempting to set a value to the wrong depth in the DictDB will raise an exception.

Example 3)
```python
test_dict3 = DictDB('test_dict3', db, value_type=int, depth=3)
test_dict3['key1']['key2']['key3'] = 1 ## ok
test_dict3['key1']['key2'] = 1 ## raise mismatch exception

test_dict2 = test_dict3['key']['key2']
test_dict2['key1'] = 1 ## ok
```

**ArrayDB(‘key’, ‘target db’, ‘return type’)**  

ArrayDB supports one dimensional array only.  
ArrayDB supports put, get, and pop. Does not support insert (adding elements in the middle of array).

```python
test_array = ArrayDB('test_array', db, value_type=int)
test_array.put(0)
test_array.put(1)
test_array.put(2)
test_array.put(3)
print(len(test_array)) ## prints 4
print(test_array.pop()) ## prints 3
test_array[0] = 0 ## ok
# test_array[100] = 1 ## error
for e in test_array: ## ok
    print(e)
print(test_array[-1]) ## ok
# print(test_array[-100]) ## error
```

#### Warning Case About VarDB, DictDB, ArrayDB
You have to Save into Block(DB) not memory.
Our platform always trust about StateDB.
so memory variable in SCORE don't allowed on Blockchain Network.
We offers Wrapping method(VarDB, ListDB, DictDB) but if you use those things wrong you would face some undifined result.

Example)  
* varDB = VarDB(...)  
varDB.set(100) (O)  
varDB = 100 (X)

* arrayDB = ArrayDB(...)  
arrayDB[0] = 100 (O)  
arrayDB = 100 (X)

* dictDB = DictDB(...)
dictDB['key0'] = 100 (O)
dictDB = 100 (X)

#### External decorator (@external)
Functions decorated with @external can be called from outside the contract.  
These functions are registered on the exportable API list.  
Any attempt to call a non-external function from outside the contract will fail.  
If a function is decorated with ‘readonly’ parameters, i.e., @external(readonly=True), 
the function will have read-only access to the state DB.  
This is similar to view keyword in Solidity.  
If the read-only external function is also decorated with @payable, the function call will fail.  
Duplicate declaration of @external will raise IconScoreException on import time.


#### Payable decorator (@payable)  
Only functions with @payable decorator are permitted to receive incoming ICX coins.  
Transferring 0 icx is acceptable.  
If ICX coins (msg.value) are passed to a non-payable function, that transaction will fail.


#### Eventlog decorator (@eventlog)  
Functions with @eventlog decorator will include logs in its TxResult as ‘eventlogs’.  
It is recommended to declare a function without implementation body.  
Even if the function has a body, it does not be executed. When declaring a function, type hinting is a must.  
Without type hinting, transaction will fail.  
The default value for the parameter can be set.

If indexed parameter is set in the decorator, designated number of parameters in the order of declaration will be indexed and included in the Bloom filter.  
At most 3 parameters can be indexed, And index can’t exceed the number of parameters(will raise an error).  
Indexed parameters and non-indexed parameters are separately stored in TxResult.

Example)
```python
# Declaration
@eventlog
def FundTransfer1(self, _backer: Address, _amount: int, _isContribution: bool): pass

@eventlog(indexed=1) # The first param (backer) will be indexed
def FundTransfer2(self, _backer: Address, _amount: int, _isContribution: bool): pass

# Execution
self.FundTransfer1(self.msg.sender, amount, True)
self.FundTransfer2(self.msg.sender, amount, True)
```

Possible data types for function parameters are primitive types (int, str, bytes, bool, Address). Array, Dictionary and None type parameter is not supported.

#### Fallback
fallback function can not be decorated with @external.  
(i.e., fallback function is not allowed to be called by external contract or user.)  
This fallback function is executed whenever the contract receives plain icx coins without data.  
If the fallback function is not decorated with @payable, the icx coin transfers to the contract will fail.


#### InterfaceScore
InterfaceScore is an interface class used to invoke other SCORE’s function.  
This interface should be used instead of legacy ‘call’ function. Usage syntax is as follows.

```python
class TokenInterface(InterfaceScore):
    @interface
    def transfer(self, addr_to: Address, value: int) -> bool: pass
```

If other SCORE has the function that has the same signature as defined here with @interface decorator, 
then that function can be invoked via InterfaceScore class object.  
Like @eventlog decorator, it is recommended to declare a function without implementation body.  
If there is a function body, it will be simply ignored.

Example) You need to get an InterfaceScore object by using IconScoreBase’s built-in function create_interface_score('score address', 'interface class').  
Using the object, you can invoke other SCORE’s external function as if it is a local function call.
```python
#Crowdsale SCORE
...
@payable
    def fallback(self):
        """
        Called when anyone sends funds to the SCORE.
        This SCORE regards it as a contribution.
        """
        if self._crowdsale_closed.get():
            revert('Crowdsale is closed.')

        # Accepts the contribution
        amount = self.msg.value
        self._balances[self.msg.sender] = self._balances[self.msg.sender] + amount
        self._amount_raised.set(self._amount_raised.get() + amount)
        value = int(amount / self._price.get())
        data = b'called from Crowdsale'

        # Gives tokens to the contributor as a reward
        token_score = self.create_interface_score(self._addr_token_score.get(), TokenInterface) # InterfaceScore define
        token_score.transfer(self.msg.sender, value, data) # InterfaceScore call

        if self.msg.sender not in self._joiner_list:
            self._joiner_list.put(self.msg.sender)

        self.FundTransfer(self.msg.sender, amount, True)
        Logger.debug(f'FundTransfer({self.msg.sender}, {amount}, True)', TAG)
```

#### Transfer ICX
Using "icx" object and it offers 2 functions "send", "transfer"

* **transfer** : Transfers designated amount of icx coin to addr_to.  
If exception occurs during execution, the exception will be escalated.  
Returns True if coin transfer succeeds.

* **send** : Sends designated amount of icx coin to addr_to.  
Basic behavior is same as transfer, the difference is that exception is caught inside the function.  
Returns True when coin transfer succeeded, False when failed.

## Let's deploy SampleToken on LocalNodeEmulator
### WARNING! 
**This time, We are going to handle about that is offered local node emulator by T-Bears.**  
**So you don't need to set your address, icx, signing and signature.**  
**If you want to deploy on Mainnet or Testnet, You have to edit URL, keyStore in tbears_cli_config.json.**

### 1. Follow and Setup T-Bears [t-bears-tutorial](https://github.com/icon-project/t-bears)

### 2. Follow and Make Sample projects(SampleToken) [score-by-example](https://github.com/icon-project/documentation/blob/master/score/score-by-example.md)

### 3. Transfer ICX A to B
**You can get your token balance is 0 now.**  
**How about your icx coin balance?**  
**Let's Query your icx balance using T-Bears**

1 - Query your icx balance.

```console
$ tbears balance hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6
```
``` console
balance in hex: 0x0
balance in decimal: 0
```

2 - Send icx to this Address(hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6) from God Address(hx0000000000000000000000000000000000000000) to 2 icx

```console
$ tbears transfer -f hx0000000000000000000000000000000000000000 hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6 2000000000000000000
```
```console
Send transfer request successfully.
transaction hash: 0x5cd000a719e4e8c3d33c5f6e47a778411a60d4d077883b51f6422c1ee776bdd4
```
```console
$ tbears txresult 0x5cd000a719e4e8c3d33c5f6e47a778411a60d4d077883b51f6422c1ee776bdd4
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x5cd000a719e4e8c3d33c5f6e47a778411a60d4d077883b51f6422c1ee776bdd4",
        "blockHeight": "0xf1a",
        "blockHash": "0x159e830bad73e2988d77f18c497de72f093acc0234eb59b8d4210a9299e0d0c6",
        "txIndex": "0x0",
        "to": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
        "stepUsed": "0xf4240",
        "stepPrice": "0x0",
        "cumulativeStepUsed": "0xf4240",
        "eventLogs": [],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

3 - Query your icx balance again

```console
$ tbears balance hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6
```
```console
balance in hex: 0x1BC16D674EC80000
balance in decimal: 2000000000000000000
```

### 4. Deploy SampleToken on Local-Node

1 - Generate T-Bears cli config
```console
$ tbears genconf
```
2 - T-Bears start emulator
```console
$ tbears start
```

3 - Open "tbears_cli_config.json" and edit like this.

**Set params in on_install**  
**__initialSupply: 1000**  
**__decimals: 18**  

```json
# tbears_cli_config.json
{
    "uri": "http://127.0.0.1:9000/api/v3",
    "nid": "0x3",
    "keyStore": null,
    "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
    "to": "cx0000000000000000000000000000000000000000",
    "deploy": {
        "stepLimit": "0x10000000",
        "mode": "install",
        "scoreParams": {
	      "_initialSupply": "0x3e8",
	      "_decimals": "0x12"
	    }
    },
    "txresult": {},
    "transfer": {
        "stepLimit": "0xf4240"
    }
}
```

4 - Deploy SampleToken
```console
$ tbears deploy sample_token -c tbears_cli_config.json
```

after that, you can get txHash like this.
```console
Send deploy request successfully.
If you want to check SCORE deployed successfully, execute txresult command
transaction hash: 0xea834af48150189b4021b9a161d4c0aff3d983ccc47ddc189bac50f55bf580b7
```

5 - Get transaction result by transaction hash.
```console
$ tbears txresult 0xea834af48150189b4021b9a161d4c0aff3d983ccc47ddc189bac50f55bf580b7
```

after that, you can get txHash like this.
```console
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0xea834af48150189b4021b9a161d4c0aff3d983ccc47ddc189bac50f55bf580b7",
        "blockHeight": "0xbe",
        "blockHash": "0xdc85bccb32109da6cfe5bb5308121ce68a1494c499d9dd8c724a0bd7397d0729",
        "txIndex": "0x0",
        "to": "cx0000000000000000000000000000000000000000",
        "scoreAddress": "cx0841205d73b93aec1062877d8d4d5ea54c6665bb",
        "stepUsed": "0x2cc8f10",
        "stepPrice": "0x0",
        "cumulativeStepUsed": "0x2cc8f10",
        "eventLogs": [],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

### 5. Handling Deployed SCORES -1

1 - Make json file to call totalSupply on Sampletoken like this.
```json
# sand.json
{
  "jsonrpc": "2.0",
  "method": "icx_call",
  "params": {
    "from": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx0841205d73b93aec1062877d8d4d5ea54c6665bb",
    "dataType": "call",
    "data": {
      "method": "totalSupply"
    }
  },
  "id": 1
}
```
```console
$ tbears call send.json
```

```console
response : {
    "jsonrpc": "2.0",
    "result": "0x3635c9adc5dea00000",
    "id": 1
}
```

2 - Edit json file to call balanceOf on Sampletoken like this.
```json
# sand.json
{
  "jsonrpc": "2.0",
  "method": "icx_call",
  "params": {
    "from": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx0841205d73b93aec1062877d8d4d5ea54c6665bb",
    "dataType": "call",
    "data": {
      "method": "balanceOf",
      "params": {
        "_owner": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6"
      }
    }
  },
  "id": 1
}
```

```console
$ tbears call send.json
```

```console
response : {
    "jsonrpc": "2.0",
    "result": "0x0",
    "id": 1
}
```



### 5. Handling Deployed SCORES -2

1 - Let's transfer this token(1 token) A to B  
This time,  Sampletoken's owner(hxe7af5fcfd8dfc67530a01a0e403882687528dfcb) transfer 1 token to hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6
```json
# send.json
{
  "jsonrpc": "2.0",
  "method": "icx_sendTransaction",
  "params": {
    "version": "0x3",
    "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
    "value": "0x0",
    "stepLimit": "0x3000000",
    "timestamp": "0x573117f1d6568",
    "nid": "0x3",
    "nonce": "0x1",
    "to": "cx658e956f66a1449212132a750d716cec418e6193",
    "signature": "",
    "dataType": "call",
    "data": {
      "method": "transfer",
      "params": {
        "_to": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
        "_value": "0xDE0B6B3A7640000"
      }
    }
  },
  "id": 1
}
```

```console
$ tbears call send.json
```

```console
response : {
    "jsonrpc": "2.0",
    "result": "0x41bf7b9ada89eb938ee4e36fce02ec86b3e68c1ceefd61decfe1e3dcc7df43a5",
    "id": 1
}
```

**You can check about eventLogs and logsBloom**  
```console
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x41bf7b9ada89eb938ee4e36fce02ec86b3e68c1ceefd61decfe1e3dcc7df43a5",
        "blockHeight": "0x4c",
        "blockHash": "0x7f53737f9ead2a04afe72d90bef072d49fa74c1f37b4029801e787aa15d37895",
        "txIndex": "0x0",
        "to": "cx658e956f66a1449212132a750d716cec418e6193",
        "stepUsed": "0xfdb2e",
        "stepPrice": "0x0",
        "cumulativeStepUsed": "0xfdb2e",
        "eventLogs": [
            {
                "scoreAddress": "cx658e956f66a1449212132a750d716cec418e6193",
                "indexed": [
                    "Transfer(Address,Address,int,bytes)",
                    "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
                    "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
                    "0xde0b6b3a7640000"
                ],
                "data": [
                    "0x4e6f6e65"
                ]
            }
        ],
        "logsBloom": "0x00000000000000100000002000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000100000040000000000000000200000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000000000000100000002000000000000000100000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000080000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

2 - Edit json file to call balanceOf on Sampletoken about hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6 again
```json
# sand.json
{
  "jsonrpc": "2.0",
  "method": "icx_call",
  "params": {
    "from": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx658e956f66a1449212132a750d716cec418e6193",
    "dataType": "call",
    "data": {
      "method": "balanceOf",
      "params": {
        "_owner": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6"
      }
    }
  },
  "id": 1
}
```

```console
$ tbears call send.json
```

```console
response : {
    "jsonrpc": "2.0",
    "result": "0xde0b6b3a7640000",
    "id": 1
}
```

## Limitations
The maximum limit of the total count of call, interface call and ICX transfer/send is 1024 in one transaction.  
The limitation of stack size increment by calling external SCORE is 64 in one transaction.  
Declaring member variables which not managed by states is prohibited.  

## API reference
https://icon-project.github.io/score-guide/api-references.html


## Summary

## Tips or FAQs

## References


