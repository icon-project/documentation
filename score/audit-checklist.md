---
title: "Audit CheckList"
---

## Overview

Described about How to audit SCORE.  
This document offers "Severity level", "Critical", "Warning"

## Intended Audience

* Mid  
* Experienced

## Purpose 

You can understand how to audit SCORE

## Prerequisite 

* [score-overview](https://github.com/icon-project/documentation/blob/master/score/score-overview.md)
* [score-writing-score](https://github.com/icon-project/documentation/blob/master/score/writing-score.md)

## Severity level

### Critical
- [Timeout](#timeout)
- [Unfinishing Loop](#unfinishing-loop)
- [Package import](#package-import)
- [System Call](#system-call)
- [Outbound Network Call](#outbound-network-call)
- [iconservice Internal API](#iconservice-internal-api)
- [Randomness](#randomness)
- [Fixed SCORE Infomation](#fixed-score-infomation)
- [IRC2 Token Standard Compliance](#irc2-token-standard-compliance)
- [IRC2 Token Parameter Name](#irc2-token-parameter-name)
- [Eventlog on Token Transfer](#eventlog-on-token-transfer)
- [Eventlog without Token Transfer](#eventlog-without-token-transfer)
- [ICXTransfer Eventlog](#icxtransfer-eventlog)
- [Super Class](#super-class)
- [Keyword Argument](#keyword-argument)
- [Big Number Operation](#big-number-operation)
- [Instance Variable](#instance-variable)
- [StateDB Operation](#statedb-operation)
- [StateDB Write Operation](#statedb-write-operation)
- [Temporary Limitation](#temporary-limitation)

### Warning
- [External Function Parameter Check](#external-function-parameter-check)
- [Internal Function Parameter Check](#internal-function-parameter-check)
- [Predictable Arbitrarity](#predictable-arbitrarity)
- [Unchecked Low Level Calls](#unchecked-low-level-calls)
- [Underflow/Overflow](#underflowoverflow)
- [Vault](#vault)
- [Reentrancy](#reentrancy)


## Critical

### Timeout

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

### Unfinishing Loop

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

### Package import

SCORE must run in a sandboxed environment.
Package import is prohibited except `iconservice` and the files in your deployed SCORE folder tree.

```python
# Bad
import os

# Good
from iconservice import *
from .myclass import *
```

### System Call

System call is prohibited.  
SCORE can not access any system resources.

```python
# Bad
import os
os.uname()
```

### Outbound Network Call

Outbound network call is prohibited.  
Outcome of network call from each node can be different.

```python
# Bad
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(host, port)
```

### iconservice Internal API

Among the API provided by ICONService, API written for platform should not be used in SCORE. Attempts to access internal APIs with information obtained using getAttr() are prohibited. Use externally released APIs only. You can find recent released APIs at https://iconservice.readthedocs.io/en/latest/.

```python 
# Bad
something = getattr(self, 'something', '')
```

### Randomness

Execution result of SCORE must be deterministic.  
Unless, nodes can not reach a consensus.
If nodes fail to reach a consensus due to the undeterministic outcomes, every transactions in the block will be lost.
Therefore, not only random function, but any attempt to prevent block generation by undeterministic operation is strictly prohibited.

```python
# Bad
# each node may have different outcome
won = datetime.datetime.now() % 2 == 0
```

### Fixed SCORE Infomation
SCORE's critical information can not be changed since it was first installed. In case of IRC2 token SCORE, name, sysmbol and decimals should not be changed; for other SCORE, name must not be changed.
```python
@external(readonly=True)
def name(self) -> str:
    return self._name.get()
#Bad
@external
def setname(self, new_name):
    self._name.set(new_name)
```
The above rules should also be guaranteed for SCORE update deployments.
For IRC2 tokens, you must update the IRC2 token SCORE with the same name, symbol and decimals value. 
If it is not an IRC2 token, it should be update to non-IRC2 SCORE with the same name value.


### IRC2 Token Standard Compliance

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

### IRC2 Token Parameter Name

When implementing IRC2 compliant token, make the parameter names in the function remain the same
as defined in [IRC2 ICON Token Standard](https://github.com/icon-project/IIPs/blob/master/IIPS/iip-2.md).

```python
# Bad
def balanceOf(self, owner: Address) -> int:

# Good
def balanceOf(self, _owner: Address) -> int:
```

### Eventlog on Token Transfer

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

### Eventlog without Token Transfer

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

### ICXTransfer Eventlog

ICXTransfer Eventlog is reserved for ICX transfer. Do not implement the Eventlog with the same name.

```python
# Bad
@eventlog(indexed=3)
def ICXTransfer(self, _from: Address, _to: Address, _value: int):
```

### Super Class

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

### Keyword Argument
Keyword argument is not allowed on_install() and on_update() function parameters

```python
# Good
def on_install(self, name: str, symbol: str, amount: int, decimals: int):
    ...

# Bad
def on_install(self, **kwargs) -> None:
    ...
```


### Big Number Operation

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

### Instance Variable

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

### StateDB Operation

In order not to cause an unexpected situation, VarDB, DictDB and ArrayDB should be accessed in a permitted manner. 

```python
# VarDB
self.test_var = VarDB('test_var', db, value_type=str)

# Good
self.test_var.set('sample')      # set
name = self.test_var.get()       # get
# Bad
self.test_var = 'sample'         # Error

# DictDB
self.test_dict = DictDB('test_dict', db, value_type=int)

# Good
self.test_dict['key'] = 1        ## set
print(self.test_dict['key'])     ## get 
# Bad
self.test_dict = 1               ## Error 

# ArrayDB
self.test_array = ArrayDB('test_array', db, value_type=int)

# Good
self.test_array.put(0)           # put the value at the last index
self.test_array.pop()            # remove the value at the last index and return it
self.test_array.get(0)           # get the value at some index
# Bad
self.test_array = 1              # Error 
```

Also, if you want to store a class instance in StateDB, you must explicitly serialize it before saving.

```python
self.something = VarDB('context.something', db, str)

# Good
self.something.set( str(Something()) )

# Bad
self.something.set( Something() )
```

### StateDB Write Operation

StateDB write operations inside of \_\_init\_\_() function is prohibited. Updating state DB in \_\_init\_\_() may cause unexpected behavior.

```python
# Bad
def __init__(self, db: IconScoreDatabase) -> None:
    super().__init__(db)
    self._total_supply = VarDB(self._TOTAL_SUPPLY, db, value_type=int)
    self._total_supply.set(10000000)
```

### Temporary Limitation
Due to the known issue of ArrayDB, declaring ArrayDB as a class member variable in __init__() may not work as intended. Following workaround is needed. ArrayDB instance must be initialized every time it is used.

```python
# Problematic (Original Usage)
def __init__(self, db: IconScoreDatabase) -> None:
    super().__init__(db)
    self.test_array = ArrayDB('test_array', db, value_type=int)

def func(self) -> None:
    self.test_array.put(0)


# Good (Temporary)
@property
def test_array(self) -> ArrayDB:
    return ArrayDB('test_array', db, value_type=int)

def __init__(self, db: IconScoreDatabase) -> None:
    super().__init__(db)
    # no declaration

def func(self) -> None:
    self.test_array.put(0)
```


## Warning

### External Function Parameter Check

If a SCORE function is called from EOA with wrong parameter types or without required parameters, ICON service will return an error.
Developers do not need to deliberately verify the input parameter types inside a function.

### Internal Function Parameter Check

If a SCORE calls other functions of own or of other SCORE, always make sure that parameter types are correct and required parameters are not omitted.
Values of parameters must be in a valid range.
There is no size limit in `str` or `int` type in Python, however, transaction message should not exceed 512 KB.
```python
# Function declarations
def myTransfer(_value: int) -> bool
def myTransfer1(_value: int, _extra: str) -> bool:

# Bad
myTransfer("1000")
myTransfer1(1000)

# Good
myTransfer(1000)
myTransfer1(1000, 'abc')
```

### Predictable arbitrarity

Some applications such as lottery require arbitrarity.  
Due to the nature of blockchain, implementation of such business logic must be done with great care.
Output of pseudo random number generator can be predictable if random seed is revealed.

```python
# Bad
# block height is predictable.
won = block.height % 2 == 0
```

### Unchecked Low Level Calls

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


### Underflow/Overflow

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

### Vault

Anybody can view the data stored in public blockchain network.  
It is strongly recommended to save personal data such as password off the blockchain network even if it is encrypted.

```python
# Bad
def change_password(self, _account: Address, _password: str):
    if self.msg.sender != _account:
        self.revert('Only owner of the account can change password')

    self.passwords[_account] = _password
```

### Reentrancy

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
