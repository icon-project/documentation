---
title: "How to use preptools for TestNet"
excerpt: "The source code is found on GitHub at https://github.com/icon-project/preptools"
---

This document is a guideline detailing how to use preptools in TestNet for P-Reps.   
P-Reps are the consensus nodes that produce, verify blocks and participate in network policy decisions on the ICON Network. 

## Intended Audience
We recommend all P-Rep candidates to go through this guideline and participate in the block generation test.

## Pre-requisites
### Requirements
* OS: MacOS, Linux
  * Windows is not supported yet.
* Python
  * In the future, we will only support 3.7.x. ** Please upgrade python version to 3.7.x. ** 
  * Optional: We recommend to create an isolated Python 3 virtual environment [virtualenv](https://virtualenv.pypa.io/en/stable/).
```bash
$ python -m venv venv             # Create a virtual environment.
$ source venv/bin/activate        # Enter the virtual environment.
```

### Installation
Install the P-Rep management command line tools
```bash
(venv) $ pip install preptools
```

## How to connect to TestNet
In order to connect to TestNet, you have to specify TestNet address when use preptools.

There are two ways to specify TestNet address:

* Using command line
    * specify url by command line
```bash
(venv) $ preptools [command] -n 1 -u "https://zicon.net.solidwallet.io" -k KEYSTORE_PATH ...
```

* Using configuration 
    * specify url by configuration
    * you can generate configure file using [genconf](#generate-configuration) command.
```bash
(venv) $ cat preptools_config.json
{
    "url": "https://zicon.net.solidwallet.io",
    "nid": 1,
    "keystore": "KEYSTORE_PATH"
} 
(venv) $ preptools [command] ...

```



## How to use preptools in TestNet
We will use the configure file to specify required information and proceed the following tutorial.

### Register P-Rep
There are three ways to Register as a P-Rep; using json file, optional argument, and interactive command line.
```bash
(venv) $ cat registerPRep.json
{
    "name": "banana node",
    "country": "USA",
    "city": "New York",
    "email": "banana@example.com",
    "website": "https://icon.banana.com",
    "details": "https://icon.banana.com/json",
    "p2pEndpoint": "node.example.com:7100"
}

# Using json file
(venv) $ preptools registerPRep --prep-json registerPRep.json

# Using optional argument and interactive command line
(venv) $ preptools registerPRep --name "kokoa node" --email "kokoa@example.com"
> Password: 
 > country : USA
 > city : New York
 > website : https://icon.kokoa.com
 > details : https://icon.kokoa.com/json
 > p2pEndpoint : node.example.com:7100
[Request] ======================================================================
{
    "from_": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx0000000000000000000000000000000000000000",
    "value": 2000000000000000000000,
    "step_limit": 268435456,
    "nid": 3,
    "nonce": null,
    "version": 3,
    "timestamp": null,
    "method": "registerPRep",
    "data_type": "call",
    "params": {
        "name": "kokoa node",
        "email": "kokoa@example.com",
        "country": "USA",
        "city": "New York",
        "website": "https://icon.kokoa.com",
        "details": "https://icon.kokoa.com/json",
        "p2pEndpoint": "node.example.com:7100"
    }
}
> Continue? [Y/n]

request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": "0x76797f770a1fae02d956e30db6ac46885d04fbd582eccc8dc84c308f03d837ae",
    "id": 1234
}
```
<sup>*</sup>DETAILS : See [JSON Standard for P-Rep detailed information](https://github.com/icon-project/documentation/blob/develop/references/JSON-Standard-for-P-Rep-detailed-Information.md)

Further information can be found in the below document:

* [Register P-Rep using preptools](https://github.com/icon-project/preptools#preptools-registerprep)

### Unregister P-Rep
unregister P-Rep. 

```bash
(venv) $ preptools unregisterPRep
> Password: 
[Request] ======================================================================
{
    "from_": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
    "to": "cx0000000000000000000000000000000000000000",
    "value": 0,
    "step_limit": 268435456,
    "nid": 1,
    "nonce": null,
    "version": 3,
    "timestamp": null,
    "method": "unregisterPRep",
    "data_type": "call",
    "params": {}
}

> Continue? [Y/n]

request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": "0x7048ea46f83579483c560db5bcb6e17e147e4f86e6aaaf2ea35b077c04ce4491",
    "id": 1234
}
```
Further information can be found in the below document:
* [Unregister P-Rep using preptools](https://github.com/icon-project/preptools#preptools-unregisterprep)

### Set P-Rep
Update P-Rep using json file, optional argument and interactive command line.

```bash
(venv) $ cat setPRep.json
{
    "name": "kokoa node",
    "country": "USA",
    "website": "https://icon.kokoa.com"
}

# with json
(venv) $ preptools setPRep --prep-json setPRep.json

# with json and arguments
(venv) $ preptools setPRep --prep-json setPRep.json --name "banana node" --email "example@iconloop.com" -i
> Password: 
[Request] ======================================================================
{
    "from_": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx0000000000000000000000000000000000000000",
    "value": 0,
    "step_limit": 268435456,
    "nid": 3,
    "nonce": null,
    "version": 3,
    "timestamp": null,
    "method": "setPRep",
    "data_type": "call",
    "params": {
        "name": "banana node",
        "country": "USA",
        "website": "https://icon.kokoa.com",
        "email": "example@iconloop.com",
    }
}

# with json, arguments and interactive mode
(venv) $ preptools setPRep --prep-json setPRep.json --name "banana node" --email "example@iconloop.com" -i
> Password: 
 > city : 
 > details : 
 > p2pEndpoint : node.example.com:7100
[Request] ======================================================================
{
    "from_": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx0000000000000000000000000000000000000000",
    "value": 0,
    "step_limit": 268435456,
    "nid": 3,
    "nonce": null,
    "version": 3,
    "timestamp": null,
    "method": "setPRep",
    "data_type": "call",
    "params": {
        "name": "banana node",
        "country": "USA",
        "website": "https://icon.kokoa.com",
        "email": "example@iconloop.com",
        "p2pEndpoint": "node.example.com:7100"
    }
}

request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": "0x7048ea46f83579483c560db5bcb6e17e147e4fas2db53d2bcdb0erf1d23b45a6b",
    "id": 1234
}
```

<sup>*</sup>DETAILS : See [JSON Standard for P-Rep detailed information](https://github.com/icon-project/documentation/blob/develop/references/JSON-Standard-for-P-Rep-detailed-Information.md)

Further information can be found in the document below:

* [Set P-Rep using preptools](https://github.com/icon-project/preptools#preptools-setprep)

### Set Governance Variables
Change Governance variables used in network operation.

```bash
(venv) $ preptools setGovernanceVariables --irep 50_000_000_000_000_000_000_000
[Request] ======================================================================
{
    "from_": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx0000000000000000000000000000000000000000",
    "value": 0,
    "step_limit": 268435456,
    "nid": 3,
    "nonce": null,
    "version": 3,
    "timestamp": null,
    "method": "setGovernanceVariables",
    "data_type": "call",
    "params": {
        "irep": "0xa968163f0a57b400000"
    }
}

> Continue? [Y/n]
request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": "0xc15b6989cd39e01b3d4bb65b72e6f7fcbc009020779b7f9fc60d59da4df7b091",
    "id": 1234
}
```
Further information can be found in the document below:
* [Set Governance Variables using preptools](https://github.com/icon-project/preptools#preptools-setgovernancevariables)

### Get P-Rep
Inquire P-Rep information

```bash
(venv) $ preptools getPRep hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6
[Request] ======================================================================
{
    "from_": "hx1234567890123456789012345678901234567890",
    "to": "cx0000000000000000000000000000000000000000",
    "method": "getPRep",
    "params": {
        "address": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6"
    }
}

request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": {
        "status": "0x1",
        "grade": "0x2",
        "name": "kokoa node",
        "country": "USA",
        "city": "New York",
        "stake": "0x0",
        "delegated": "0x0",
        "totalBlocks": "0x0",
        "validatedBlocks": "0x0",
        "irep": "0xa968163f0a57b400000",
        "irepUpdateBlockHeight": "0x58f",
        "lastGenerateBlockHeight": "0x1aa",
        "email": "example@iconloop.com",
        "website": "https://icon.kokoa.com",
        "details": "https://icon.kokoa.com/json",
        "p2pEndpoint": "node.example.com:7100",
        "publicKey": "0x040d60ccc4fd29307304a8e84715e6e1a2e643bcff14fbf90d9099dfc84585a6f6f0b6944594efebe433a12a005ba56d215d6e51697a3360b5d741f8db89955c66"
    },
    "id": 1234
}
```
Further information can be found in the document below:
* [Get P-Rep using preptools](https://github.com/icon-project/preptools#preptools-getprep)

### Get P-Reps
Get live status of all registered P-Rep candidates

``` bash
(IS-769) rhkddnjs99-1:test_env gwangwon-choi-mac$ preptools getPReps
[Request] ======================================================================
{
    "from_": "hx1234567890123456789012345678901234567890",
    "to": "cx0000000000000000000000000000000000000000",
    "method": "getPReps",
    "params": null
}

request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": {
        "blockHeight": "0xf",
        "startRanking": "0x1",
        "totalDelegated": "0x9df35e3b466ea89dac00000",
        "totalStake": "0x14af1b93f3e3d30875000000",
        "preps": [
            {
                "status": "0x0",
                "grade": "0x2",
                "name": "nodehx000e0415037ae871184b2c7154e5924ef2bc075e",
                "country": "KOR",
                "city": "Unknown",
                "stake": "0xf0afcc8b15fdf4bf800000",
                "delegated": "0x7857e6458afefa5fc00000",
                "totalBlocks": "0x2",
                "validatedBlocks": "0x2",
                "irep": "0xa968163f0a57b400000",
                "irepUpdateBlockHeight": "0x5",
                "lastGenerateBlockHeight": "0x1eb",
                "address": "hx000e0415037ae871184b2c7154e5924ef2bc075e"
            },
            ... ,
            ... ,
            {
                "status": "0x0",
                "grade": "0x2",
                "name": "nodehx9bb77b43ff288395cee8f861376ec7421c6390f3",
                "country": "KOR",
                "city": "Unknown",
                "stake": "0xf0afcc8b15fdf4bf800000",
                "delegated": "0x7857e6458afefa5fc00000",
                "totalBlocks": "0x2",
                "validatedBlocks": "0x2",
                "irep": "0xa968163f0a57b400000",
                "irepUpdateBlockHeight": "0x5",
                "lastGenerateBlockHeight": "0x1ba",
                "address": "hx9bb77b43ff288395cee8f861376ec7421c6390f3"
            }
        ]
    },
    "id": 1234
}
```
Further information can be found in the document below:
* [Get P-Reps using preptools](https://github.com/icon-project/preptools#preptools-getpreps)

### Get Transaction Result
Get Transaction result by transaction hash
```bash
(venv) $ preptools txresult 0xc8456053128897a0941dab4c79428db91dda5a2899e3813698146ac25808c4c9
request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0xc8456053128897a0941dab4c79428db91dda5a2899e3813698146ac25808c4c9",
        "blockHeight": "0x1d980",
        "blockHash": "0xa7354fb9427308239e56482916dd4e31988ce1f207091cdf81c656f89a066c5f",
        "txIndex": "0x0",
        "to": "cx0000000000000000000000000000000000000000",
        "stepUsed": "0x21340",
        "stepPrice": "0x2540be400",
        "cumulativeStepUsed": "0x21340",
        "eventLogs": [
            {
                "scoreAddress": "cx0000000000000000000000000000000000000000",
                "indexed": [
                    "PRepSet(Address)"
                ],
                "data": [
                    "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6"
                ]
            }
        ],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000008000000000000000000000000080000000000000000000000000000000000000000000000000000000000020000000000000008000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1234
}
```
Further information can be found in the document below:
* [Get Transaction Result using preptools](https://github.com/icon-project/preptools#preptools-txresult)


### Get Transaction
Get Transaction information by transaction hash
```bash
(venv) $ preptools txbyhash 0xc8456053128897a0941dab4c79428db91dda5a2899e3813698146ac25808c4c9
request success.
[Response] =====================================================================
{
    "jsonrpc": "2.0",
    "result": {
        "version": "0x3",
        "from": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
        "to": "cx0000000000000000000000000000000000000000",
        "stepLimit": "0x10000000",
        "timestamp": "0x58fa97a48ad43",
        "nid": "0x3",
        "value": "0x0",
        "dataType": "call",
        "data": {
            "method": "setPRep",
            "params": {
                "name": "kokoa node",
                "country": "USA",
                "website": "https://icon.kokoa.com",
                "details": "https://icon.kokoa.com/json",
                "p2pEndpoint": "node.example.com:7100"
            }
        },
        "signature": "4krblW9KtQr6KNIJOVa22B3JFDQD6vaxepDSjMET91oua3Qeiq3UFMIHiucWiIrKGt3zaSo2K+mVW7Ge5rOtPwE=",
        "txHash": "0xc8456053128897a0941dab4c79428db91dda5a2899e3813698146ac25808c4c9",
        "txIndex": "0x0",
        "blockHeight": "0x1d980",
        "blockHash": "0xa7354fb9427308239e56482916dd4e31988ce1f207091cdf81c656f89a066c5f"
    },
    "id": 1234
}
```
Further information can be found in the document below:
* [Get Transaction using preptools](https://github.com/icon-project/preptools#tbears-txbyhash)

### Generate configuration
Generate preptools_config.json in current folder
```bash
(venv) $ preptools genconf
Made ./preptools_config.json successfully

(venv) $ cat preptools_config.json 
{
    "url": "http://127.0.0.1:9000/api/v3",
    "nid": 3,
    "keystore": null
}
```

Further information can be found in the document below:

- [Generate configure file using preptools](https://github.com/icon-project/preptools#preptools-genconf)

### Generate keystore

Generate keystore file with specified path
```bash
(venv) $ preptools keystore prep_keystore
Input your keystore password: 
Retype your keystore password: 
Made keystore file successfully
```

Further information can be found in the document below:

- [Generate keystore using preptools](https://github.com/icon-project/preptools#preptools-keystore)

## License

This project follows the Apache 2.0 License. Please refer to [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) for details.
