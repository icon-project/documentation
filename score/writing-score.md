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

* [score-overview](https://github.com/icon-project/documentation/blob/develop/score/score-overview.md)

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
token_score = self.create_interface_score(self._addr_token_score.get(), TokenInterface)
token_score.transfer(self.msg.sender, value)
```

#### Transfer ICX
Using "icx" object and it offers 2 functions "send", "transfer"

* **transfer** : Transfers designated amount of icx coin to addr_to.  
If exception occurs during execution, the exception will be escalated.  
Returns True if coin transfer succeeds.

* **send** : Sends designated amount of icx coin to addr_to.  
Basic behavior is same as transfer, the difference is that exception is caught inside the function.  
Returns True when coin transfer succeeded, False when failed.  

## Limitations
The maximum limit of the total count of call, interface call and ICX transfer/send is 1024 in one transaction.  
The limitation of stack size increment by calling external SCORE is 64 in one transaction.  
Declaring member variables which not managed by states is prohibited.  

## API reference
https://icon-project.github.io/score-guide/api-references.html

## Audit Checklist

### Severity level

#### Critical
- [Timeout](#timeout)
- [Unfinishing loop](#unfinishing-loop)
- [Package import](#package-import)
- [System call](#system-call)
- [Randomness](#randomness)
- [Outbound network call](#outbound-network-call)
- [IRC2 Token Standard compliance](#irc2-token-standard-compliance)
- [IRC2 Token parameter name](#irc2-token-parameter-name)
- [Eventlog on Token Transfer](#eventlog-on-token-transfer)
- [Eventlog without Token Transfer](#eventlog-without-token-transfer)
- [ICXTransfer Eventlog](#icxtransfer-eventlog)
- [Big Number Operation](#big-number-operation)
- [Instance Variable](#instance-variable)

#### Warning
- [External Function Parameter Check](#external-function-parameter-check)
- [Internal Function Parameter Check](#internal-function-parameter-check)
- [Predictable arbitrarity](#predictable-arbitrarity)
- [Unchecked Low Level Calls](#unchecked-low-level-calls)
- [Super Class](#super-class)
- [Underflow/Overflow](#underflowoverflow)
- [Vault](#vault)
- [Reentrancy](#reentrancy)


### Critical

#### Timeout

SCORE function must return fairly immediately.  
Blockchain is not for any long-running operation.
For example, if you implement token airdrop to many users, do not iterate over all users in a single function.  
Handle each or partial airdrop(s) one by one instead.

```python
# Bad
@external
def airdrop_token(self, _value: int, _data: bytes = None):
    for target in self._very_large_targets:
        self._transfer(self.msg.sender, target, _value, _data)

# Good
@external
def airdrop_token(self, _to: Address, _value: int, _data: bytes = None):
    if self._airdrop_sent_address[_to]:
        self.revert(f"Token was dropped already: {_to}")
    self._airdrop_sent_address[_to] = True
    self._transfer(self.msg.sender, _to, _value, _data)
```

#### Unfinishing loop

Use `for` and `while` statement carefully. Make sure that the code always reaches the exit condition.
If the operation inside the loop consumes `step`, the program will halt at some point.
However, if the code block inside the loop does not consume `step`,
i.e., Python built-in functions, then the program may hang there forever.
ICON network will force-kill the hanging task, but it may still degrade significantly the ICON network.

```python
# Bad
while True:
    # do something without consuming 'step' or proper exit condition

# Good
i = 0
while i < 10:
    # do something
    i += 1
```

#### Package import

SCORE must run in a sandboxed environment.
Package import is prohibited except `iconservice` and the files in your deployed SCORE folder tree.

```python
# Bad
import os

# Good
from iconservice import *
from .myclass import *
```

#### System call

System call is prohibited.  
SCORE can not access any system resources.

```python
# Bad
import os
os.uname()
```

#### Randomness

Execution result of SCORE must be deterministic.  
Unless, nodes can not reach a consensus.
If nodes fail to reach a consensus due to the undeterministic outcomes, every transactions in the block will be lost.
Therefore, not only random function, but any attempt to prevent block generation by undeterministic operation is strictly prohibited.

```python
# Bad
# each node may have different outcome
won = datetime.datetime.now() % 2 == 0
```

#### Outbound network call

Outbound network call is prohibited.  
Outcome of network call from each node can be different.

```python
# Bad
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(host, port)
```

#### IRC2 Token Standard compliance

IRC2 compliant token must implement every functions in the specification.  
[IRC2 ICON Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md)

```python
# IRC2 functions
@external(readonly=True)
def name(self) -> str:

@external(readonly=True)
def symbol(self) -> str:

@external(readonly=True)
def decimals(self) -> int:

@external(readonly=True)
def totalSupply(self) -> int:

@external(readonly=True)
def balanceOf(self, _owner: Address) -> int:

@external
def transfer(self, _to: Address, _value: int, _data: bytes=None):
```

#### IRC2 Token parameter name

When implementing IRC2 compliant token, make the parameter names in the function remain the same
as defined in [IRC2 ICON Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md).

```python
# Bad
def balanceOf(self, owner: Address) -> int:

# Good
def balanceOf(self, _owner: Address) -> int:
```

#### Eventlog on Token Transfer

Token transfer must trigger Eventlog.

```python
# Good
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _value: int, _data: bytes):
    pass

@external
def transfer(self, _to: Address, _value: int, _data: bytes = None):
    self._balances[self.msg.sender] -= _value
    self._balances[_to] += _value
    self.Transfer(self.msg.sender, _to, _value, _data)
```

#### Eventlog without Token Transfer

Do not trigger Transfer Eventlog without token transfer.

```python
# Bad
@eventlog(indexed=3)
def Transfer(self, _from: Address, _to: Address, _value: int, _data: bytes):
    pass

@external
def doSomething(self, _to: Address, _value: int):
    #  no token transfer occurred
    self.Transfer(self.msg.sender, _to, _value, None)
```

#### ICXTransfer Eventlog

ICXTransfer Eventlog is reserved for ICX transfer. Do not implement the Eventlog with the same name.

```python
# Bad
@eventlog(indexed=3)
def ICXTransfer(self, _from: Address, _to: Address, _value: int):
```

#### Big Number Operation

The maximum result of a numeric operation must be less than (2<sup>256</sup> - 1).  
If the result is bigger than (2<sup>256</sup> - 1), the python interpreter can not perform the operation.  
To avoid errors, you must understand that input parameters may cause errors, and validate if the number is within the range.

```python
# Bad (on_install params - decimal value is 1_000_000_000_000_000_000)
{
	"contentType": "application/zip",
	"params": {
		"initialSupply": "0x2540be400",
		"decimals": "0xde0b6b3a7640000"
	}
}

# Good (on_install params decimal value is 18)
{
	"contentType": "application/zip",
	"params": {
		"initialSupply": "0x2540be400",
		"decimals": "0x12"
	}
}

def on_install(self, initialSupply: int, decimals: int) -> None:
    super().on_install()

    total_supply = initialSupply * 10 ** decimals
    Logger.debug(f'on_install: total_supply={total_supply}', TAG)

    self._total_supply.set(total_supply)
    self._decimals.set(decimals)
    self._balances[self.msg.sender] = total_supply
```

```python
# Bad
@external
def big_number_op(self, _value: int) -> None:
    self._result = 10 ** _value

# Good
@external
def big_number_op(self, _value: int) -> None:
    # check if _value causes the big number operation error
    if _value > 77:
        self.revert("_value is too big to operate")
    self._result = 10 ** _value
```

#### Instance Variable

On each node, the SCORE instance can be loaded/unloaded at any time.  
Therefore, if you use instance variables that are not stored in StateDB, you may have different result on each node.

```python
# Bad
def __init__(self, db: IconScoreDatabase) -> None:
    super().__init__(db)

@external
def update_organizer(self, _organizer: Address) -> None:
    self._organizer = _organizer

@external
def get_organizer(self) -> Address:
    return self._organizer

# Good
def __init__(self, db: IconScoreDatabase) -> None:
    super().__init__(db)
    self._organizer = VarDB(self._ORGANIZER, db, value_type=Address)

@external
def update_organizer(self, _organizer: Address) -> None:
    self._organizer.set(_organizer) 

@external
def get_organizer(self) -> Address:
    return self._organizer.get()
```

### Warning

#### External Function Parameter Check

If a SCORE function is called from EOA with wrong parameter types or without required parameters, ICON service will return an error.
Developers do not need to deliberately verify the input parameter types inside a function.

#### Internal Function Parameter Check

If a SCORE calls other functions of own or of other SCORE, always make sure that parameter types are correct and required parameters are not omitted.
Values of parameters must be in a valid range.
There is no size limit in `str` or `int` type in Python, however, transaction message should not exceed 512 KB.
```python
# Function declarations
def myTransfer(_value: int) -> bool:
def myTransfer1(_value: int, _extra: str) -> bool:

# Bad
myTransfer("1000")
myTransfer1(1000)

# Good
myTransfer(1000)
myTransfer1(1000, 'abc')
```

#### Predictable arbitrarity

Some applications such as lottery require arbitrarity.  
Due to the nature of blockchain, implementation of such business logic must be done with great care.
Output of pseudo random number generator can be predictable if random seed is revealed.

```python
# Bad
# block height is predictable.
won = block.height % 2 == 0
```

#### Unchecked Low Level Calls

In case of sending ICX by calling a low level function such as 'icx.send', you should check the execution result of 'icx.send' and handle the failure properly.  
'icx.send' returns boolean result of its execution, and does not raise an exception on failure.  
On the other hand, 'icx.transfer' raises an exception if transaction fails.  
If the SCORE does not catch the exception, the transaction will be reverted.  
[Reference: An object used to transfer icx coin]( https://github.com/icon-project/icon-service/blob/master/docs/dapp_guide.md#icx--an-object-used-to-transfer-icx-coin)

```python

# Bad
self._refund_icx_amount[_to] += amount
self.icx.send(_to)

# Good
self._refund_icx_amount[_to] += amount
if not self.icx.send(_to, amount):
    self._refund_icx_amount[_to] -= amount

# Good
self._refund_icx_amount[_to] += amount
self.icx.transfer(_to, amount)
```

#### Super Class

In your SCORE main class that inherits IconScoreBase, you must call super().\_\_init\_\_() in the \_\_init\_\_() function to initialize the state DB. Likewise, super().on_install() must be called in on_install() function and super().on_update() must be called in on_update() function.

```python
# Bad
class MyClass(IconScoreBase):
    def __init__(self, db: IconScoreDatabase) -> None:
        self._context__name = VarDB('context.name', db, str)
        self._context__cap = VarDB('context.cap', db, int)

    def on_install(self, name: str, cap: str) -> None:
        # doSomething

    def on_update(self) -> None:
        # doSomething

# Good
class MyClass(IconScoreBase):
    def __init__(self, db: IconScoreDatabase) -> None:
        super().__init__(db)
        self._context__name = VarDB('context.name', db, str)
        self._context__cap = VarDB('context.cap', db, int)

    def on_install(self, name: str, cap: str) -> None:
        super().on_install()
        # doSomething

    def on_update(self) -> None:
        super().on_update()
        # doSomething
```

#### Underflow/Overflow

When you do arithmetic, it is really important to validate that operands and results are in the designed range.

```python
# Bad
@external
def mint_token(self, _amount: int):
    if not self.msg.sender == self.owner:
        self.revert('Only owner can mint token')

    # if _amount is below zero, self._balances[self.owner] and self._total_supply can be minus potentially
    self._balances[self.owner] = self._balances[self.owner] + _amount
    self._total_supply.set(self._total_supply.get() + _amount)

    self.Transfer(EOA_ZERO, self.owner, _amount, b'mint')

# Good  
@external
def mint_token(self, _amount: int):
    if not self.msg.sender == self.owner:
        self.revert('Only owner can mint token')
    if _amount <= 0:
        self.revert('_amount should be greater than 0')

    self._balances[self.owner] = self._balances[self.owner] + _amount
    self._total_supply.set(self._total_supply.get() + _amount)

    self.Transfer(EOA_ZERO, self.owner, _amount, b'mint')
```

#### Vault

Anybody can view the data stored in public blockchain network.  
It is strongly recommended to save personal data such as password off the blockchain network even if it is encrypted.

```python
# Bad
def change_password(self, _account: Address, _password: str):
    if self.msg.sender != _account:
        self.revert('Only owner of the account can change password')

    self.passwords[_account] = _password
```

#### Reentrancy

When you send ICX or token, keep it mind that the target could be a SCORE.  
If the SCORE's fallback or tokenFallback function is implemented maliciously, it could reenter original SCORE.  
Then there could be unintended loop between two SCOREs.

```python
# Bad
# refund function in SCORE1. (assume ICX:token ratio is 1:1)
@external
def refund(self, _to:Address, _amount:int):
    if self.msg.sender != self.owner and self.msg.sender != _to:
        self.revert('You are not allowed to request refund')
    if self.token_balances[_to] < _amount:
        self.revert('Not enough balance')

    # send icx first
    self.icx.transfer(_to, _amount)

    # decrease balance later
    self.token_balances[_to] -= _amount

# malicious fallback function in SCORE2
@payable
def fallback(self):
    if self.msg.sender == SCORE1_ADDRESS:
        # call refund of SCORE1 Again
        score1 = self.create_interface_score(SCORE1_ADDRESS, Score1Interface)
        score1.refund(self.msg.sender, bigAmountOfICX)

# Good
# refund function in SCORE1
@external
def refund(self, _to:Address, _amount:int):
    if self.msg.sender != self.owner and self.msg.sender != _to:
        self.revert('You are not allowed to request refund')
    if self.token_balances[_to] < _amount:
        self.revert('Not enough balance')

    # decrease balance first
    self.balances[_to] -= _amount

    # send icx later
    self.icx.transfer(_to, _amount)

# Good
# refund function in SCORE1
@external
def refund(self, _to:Address, _amount:int):
    if self.msg.sender != self.owner and self.msg.sender != _to:
        self.revert('You are not allowed to request refund')
    if self.token_balances[_to] < _amount:
        self.revert('Not enough balance')

    # block if _to is smart contract
    if _to.is_contract:
        self.revert('ICX can not be transferred to SCORE')

    # send icx first
    self.icx.transfer(_to, _amount)

    # decrease balance later
    self.balances[_to] -= _amount
```

## Summary

## Tips or FAQs

## References


