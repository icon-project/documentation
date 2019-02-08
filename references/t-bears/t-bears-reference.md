---
title: "T-Bears Command Line Interface Reference Guide"
---

This page contains information about the T-Bears CLI commands and their arguments. This CLI reference guide includes details about T-Bears version 1.1.0.1.
[TOC]
## CLI Usage
```bash
usage: tbears [-h] [-v] command ...

tbears v1.1.0.1 arguments

optional arguments:
  -h, --help     show this help message and exit
  -v, --verbose  Verbose mode

Available commands:
  If you want to see help message of commands, use "tbears command -h"

  command
    start        Start tbears service
    stop         Stop tbears service
    deploy       Deploy the SCORE
    clear        Clear all SCOREs deployed on tbears service
    test         Run the unittest in the project
    init         Initialize tbears project
    samples      This command has been deprecated since v1.1.0
    genconf      Generate tbears config files. (tbears_server_config.json,
                 tbears_cli_config.json and keystore_test1)
    console      Get into tbears interactive mode by embedding IPython
    txresult     Get transaction result by transaction hash
    transfer     Transfer ICX coin.
    keystore     Create a keystore file in the specified path
    balance      Get balance of given address in loop unit
    totalsupply  Query total supply of ICX in loop unit
    scoreapi     Get score's api using given score address
    txbyhash     Get transaction by transaction hash
    lastblock    Get last block's info
    blockbyhash  Get block info using given block hash
    blockbyheight
                 Get block info using given block height
    sendtx       Request icx_sendTransaction with the specified json file and
                 keystore file. If keystore file is not given, tbears sends
                 request as it is in the json file.
    call         Request icx_call with the specified json file.
```

## CLI Commands
This chapter discusses the following sections.

 - T-Bears Service Commands
 - T-Bears SCORE Commands
 - Other ICON JSON-RPC API Commands
 - T-Bears Utility Commands

### T-Bears Service Commands
Commands that manage the T-Bears service. There are three commands `start`, `stop` and `clear`.

#### start
- Description
Start T-Bears service. Whenever T-Bears service starts, it loads the configuration from `tbears_server_config.json` file. If you want to use other configuration file, you can specify the file location with the '-c' option.
- Usage
```bash
usage: tbears start [-h] [-a HOSTADDRESS] [-p PORT] [-c CONFIG]

Start tbears service

optional arguments:
  -h, --help            show this help message and exit
  -a HOSTADDRESS, --address HOSTADDRESS
                        Address to host on (default: 0.0.0.0)
  -p PORT, --port PORT  Port to host on (default: 9000)
  -c CONFIG, --config CONFIG
                        tbears configuration file path (default:
                        ./tbears_server_config.json)
```
- Example
```bash
$ tbears start -p 9100 -c ./tbears_server_config.json
Started tbears service successfully
$ tbears start -p 9100 -c ./tbears_server_config.json
port 9100 already in use. use other port.
```

#### stop
- Description
Stop all running SCOREs and T-Bears service.
- Usage
```bash
usage: tbears stop [-h]

Stop all running SCOREs and tbears service

optional arguments:
  -h, --help  show this help message and exit
```
- Example
```bash
$ tbears stop
Stopped tbears service successfully
```

#### clear
- Description
Clear all SCOREs deployed on local T-Bears service.
- Usage
```bash
usage: tbears clear [-h]

Clear all SCOREs deployed on local tbears service

optional arguments:
  -h, --help  show this help message and exit
```
- Example
```bash
$ tbears clear
Cleared SCORE deployed on tbears successfully
```

### T-Bears SCORE Commands
These commands are related to SCORE development, test and deployment. `init` generates SCORE project template. `test` is used to run unittest in the SCORE. `deploy` is used to deploy the SCORE. `scoreapi', 'sendtx' and 'call' are used to query and call the external method of the SCORE.
#### init
- Description
Initialize SCORE development environment. Generate <project\>.py, package.json and test code in <project\> directory. The name of the SCORE class is \<scoreClass\>.  Default configuration files, `tbears_server_config.json` used when starting T-Bears and `tbears_cli_config.json` used when deploying SCORE, are also generated.
- Usage
```bash
usage: tbears init [-h] project scoreClass

Initialize SCORE development environment. Generate <project>.py, package.json
and test code in <project> directory. The name of the score class is
<scoreClass>.

positional arguments:
  project     Project name
  scoreClass  SCORE class name

optional arguments:
  -h, --help  show this help message and exit
```
- Example
```bash
$ tbears init project_test TestScore
Initialized project_test successfully
$ tbears init project_test TestScore
error: argument project: 'project_test' must be empty
...
```
- File Description
| Item                       | Description                                              |
| :------------------------- | :----------------------------------------------------------- |
| tbears_server_config.json  | T-Bears default configuration file will be created on the working directory. |
| tbears_cli_config.json     | Configuration file for CLI commands will be created on the working directory. |
| keystore_test1             | Keystore file for test account will be created on the working directory. |
| \<project>                 | SCORE project name. Project directory is created with the same name. |
| \<project>/\_\_init\_\_.py | \_\_init\_\_.py file to make the project directory recognized as a python package. |
| \<project>/package.json    | Contains the information needed when SCORE is loaded. <br> "main_file" and "main_class" is necessary. |
| \<project>/<project>.py    | SCORE main file. ABCToken class is defined.                  |
| \<project>/tests           | Directory for SCORE test code.                              |
| \<project>/tests/\_\_init\_\_.py | \_\_init\_\_.py file to make the test directory recognized as a python package. |
| \<project>/tests/test\_<project>.py    | SCORE test main file.                              |

#### test
- Description
Run the unittest in the SCORE project.
- Usage
```bash
usage: tbears test [-h] project

Run the unittest in the project

positional arguments:
  project     Project directory path

optional arguments:
  -h, --help  show this help message and exit
```
- Example
```bash
tbears test project_test/
..
----------------------------------------------------------------------
Ran 2 tests in 0.180s

OK
```

#### deploy
- Description
Deploy the SCORE. You can deploy it on local T-Bears service or on ICON network.

`tbears_cli_config.json` file contains the deployment configuration properties. If you want to use other configuration file, you can specify the file location with the '-c' option.
- Usage
```bash
usage: tbears deploy [-h] [-u URI] [-t {tbears,zip}] [-m {install,update}]
                     [-f FROM] [-o TO] [-k KEYSTORE] [-n NID] [-p PASSWORD]
                     [-s STEPLIMIT] [-c CONFIG]
                     project

Deploy the SCORE

positional arguments:
  project               Project directory path or zip file path

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -t {tbears,zip}, --type {tbears,zip}
                        This option has been deprecated since v1.0.5. Deploy
                        command supports zip type only
  -m {install,update}, --mode {install,update}
                        Deploy mode (default: install)
  -f FROM, --from FROM  From address. i.e. SCORE owner address
  -o TO, --to TO        To address. i.e. SCORE address
  -k KEYSTORE, --key-store KEYSTORE
                        Keystore file path. Used to generate "from" address
                        and transaction signature
  -n NID, --nid NID     Network ID
  -p PASSWORD, --password PASSWORD
                        keystore file's password
  -s STEPLIMIT, --step-limit STEPLIMIT
                        Step limit
  -c CONFIG, --config CONFIG
                        deploy config path (default: ./tbears_cli_config.json)
```
- Example
```bash
$ tbears deploy score_proj
error: argument project: There is no 'score_proj'
...
# deploy SCORE with keystore file
$ tbears deploy project_test/ -k keystore_test1
Input your keystore password:
Send deploy request successfully.
If you want to check SCORE deployed successfully, execute txresult command
transaction hash: 0xe5a15bdb4f54055c01c6c2b931c3d37eb2e2c09ef1a04b25b97ae328a882051a
# update SCORE with keystore file
$ tbears deploy project_test/ -m update -o cxd5dd40e1565bc97910ba97a40907d43574a05aa4 -k keystore_test1
Input your keystore password:
Send deploy request successfully.
If you want to check SCORE deployed successfully, execute txresult command
transaction hash: 0xa2e1941dd3aaf607d05aecd48af6eb8108be32bc3368fe30ea58d7dfad907551
```

#### scoreapi
- Description
Get list of APIs that the given SCORE provides. Please refer to icx_getScoreApi of [ICON JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/develop/docs/icon-json-rpc-v3.md#icx_getscoreapi) for details.
- Usage
```bash
usage: tbears scoreapi [-h] [-u URI] [-c CONFIG] address

Get score's api using given score address

positional arguments:
  address               Score address to query score api

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ tbears scoreapi hxe7af5fcfd8dfc67530a01a0e403882687528dfcb
error: argument address: Invalid address 'hxe7af5fcfd8dfc67530a01a0e403882687528dfcb'. Address must start with 'cx'
$ tbears scoreapi cxe7af5fcfd8dfc67530a01a0e403882687528dfcb
Got an error response
Can not get cxe7af5fcfd8dfc67530a01a0e403882687528dfcb's API
{
    "jsonrpc": "2.0",
    "error": {
        "code": -32602,
        "message": "SCORE is inactive: cxe7af5fcfd8dfc67530a01a0e403882687528dfcb"
    },
    "id": 1
}
$ tbears scoreapi cxd5dd40e1565bc97910ba97a40907d43574a05aa4
SCORE API: [
    {
        "type": "fallback",
        "name": "fallback",
        "inputs": []
    },
    {
        "type": "function",
        "name": "hello",
        "inputs": [],
        "outputs": [
            {
                "type": "str"
            }
        ],
        "readonly": "0x1"
    }
]
```

#### sendtx
- Description
Request `icx_sendTransaction` with the specified json file.
- Usage
```bash
usage: tbears sendtx [-h] [-u URI] [-k KEYSTORE] [-c CONFIG] [-p PASSWORD]
                     json_file

Request icx_sendTransaction with the specified json file and keystore file. If
keystore file is not given, tbears sends request as it is in the json file.

positional arguments:
  json_file             File path containing icx_sendTransaction content

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -k KEYSTORE, --key-store KEYSTORE
                        Keystore file path. Used to generate "from"address and
                        transaction signature
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
  -p PASSWORD, --password PASSWORD
                        Keystore file's password
```
- Example
```bash
$ cat send.json
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "to": "cx19c17c9c7a31f4820bf15daf0dd2e61e47db08b8",
        "value": "0x0",
        "stepLimit": "0x200000",
        "nid": "0x3",
        "nonce": "0x123461",
        "dataType": "call",
        "data": {
                "method": "transfer",
                "params": {
                        "_to": "hx08624fb0feec1a41957032c3c853e73850aedae1",
                        "_value": "0x1"
                }
        }
    }
}
$ tbears sendtx send.json -k keystore_test1
input your keystore password:
Send transaction request successfully.
transaction hash: 0xc8a3e3f77f21f8f1177d829cbc4c0ded6fd064cc8e42ef309dacff5c0a952289
```

#### call
- Description
Request `icx_call` with the specified json file.
- Usage
```bash
usage: tbears call [-h] [-u URI] [-c CONFIG] json_file

Request icx_call with the specified json file.

positional arguments:
  json_file             File path containing icx_call content

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ cat call.json
{
  "jsonrpc": "2.0",
  "method": "icx_call",
  "params": {
    "from": "hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6",
    "to": "cx53d5080a7d8a805bb10eb9bc64637809dc910832",
    "dataType": "call",
    "data": {
      "method": "hello"
    }
  },
  "id": 1
}
$ tbears call call.json
response : {
    "jsonrpc": "2.0",
    "result": "hello",
    "id": 1
}
```

### Other ICON JSON-RPC API Commands
Commands that are related to ICX coin, transaction, and block.
#### transfer
- Description
Transfer designated amount of ICX coins.
- Usage
```bash
usage: tbears transfer [-h] [-f FROM] [-k KEYSTORE] [-n NID] [-u URI]
                       [-p PASSWORD] [-s STEPLIMIT] [-c CONFIG]
                       to value

Transfer ICX coin.

positional arguments:
  to                    Recipient
  value                 Amount of ICX coin in loop to transfer (1 icx = 1e18
                        loop)

optional arguments:
  -h, --help            show this help message and exit
  -f FROM, --from FROM  From address.
  -k KEYSTORE, --key-store KEYSTORE
                        Keystore file path. Used to generate "from" address
                        and transaction signature
  -n NID, --nid NID     Network ID (default: 0x3)
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -p PASSWORD, --password PASSWORD
                        Keystore file's password
  -s STEPLIMIT, --step-limit STEPLIMIT
                        Step limit
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        values for the properties "keyStore", "uri", "from"
                        and "stepLimit". (default: ./tbears_cli_config.json)
```
- Example
```bash
$ tbears transfer -f hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa hxbbaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab 100
Got an error response
{
    "jsonrpc": "2.0",
    "error": {
        "code": -32600,
        "message": "Out of balance: balance(0) < value(100) + fee(0)"
    },
    "id": 1
}
$ tbears transfer hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa 1e18 -k keystore_test1
Input your keystore password:
Send transfer request successfully.
transaction hash: 0x6334856b5da9b2d4d841c1bbfa27fc0a0f3568d17bb9cac91302ce64f83aa6db
```

#### balance
- Description
Get balance of given address.
- Usage
```bash
usage: tbears balance [-h] [-u URI] [-c CONFIG] address

Get balance of given address in loop unit

positional arguments:
  address               Address to query the ICX balance

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ tbears balance hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
balance in hex: 0xde0b6b3a7640000
balance in decimal: 1000000000000000000
$ tbears balance hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaab
balance in hex: 0x0
balance in decimal: 0
$ tbears balance cxd5dd40e1565bc97910ba97a40907d43574a05aa4
balance in hex: 0x0
balance in decimal: 0
$ tbears balance hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
error: argument address: Invalid address 'hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa'
...
```

#### totalsupply
- Description
Query total supply of ICX.
- Usage
```bash
usage: tbears totalsupply [-h] [-u URI] [-c CONFIG]

Query total supply of ICX in loop unit

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ tbears totalsupply
Total supply of ICX in hex: 0x52c3fff19494c464f000000
Total supply of ICX in decimal: 1600920000000000000000000000
```

#### txresult
- Description
Get transaction result by transaction hash.
- Usage
```bash
usage: tbears txresult [-h] [-u URI] [-c CONFIG] hash

Get transaction result by transaction hash

positional arguments:
  hash                  Hash of the transaction to be queried.

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
# update SCORE
$ tbears txresult 0xa2e1941dd3aaf607d05aecd48af6eb8108be32bc3368fe30ea58d7dfad907551
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0xa2e1941dd3aaf607d05aecd48af6eb8108be32bc3368fe30ea58d7dfad907551",
        "blockHeight": "0x10",
        "blockHash": "0xb8f13a792808f8be5a7169d51f5562ab9913aaccb925d048cb6e4877d0659041",
        "txIndex": "0x0",
        "to": "cxd5dd40e1565bc97910ba97a40907d43574a05aa4",
        "scoreAddress": "cxd5dd40e1565bc97910ba97a40907d43574a05aa4",
        "stepUsed": "0x14f68d8",
        "stepPrice": "0x0",
        "cumulativeStepUsed": "0x14f68d8",
        "eventLogs": [],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
# transfer ICX
$ tbears txresult 0x6334856b5da9b2d4d841c1bbfa27fc0a0f3568d17bb9cac91302ce64f83aa6db
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x6334856b5da9b2d4d841c1bbfa27fc0a0f3568d17bb9cac91302ce64f83aa6db",
        "blockHeight": "0xd7",
        "blockHash": "0x59ec3abfada762374c9cfd058d1d950f5e22c989e01bcb9b15c378d980ed83aa",
        "txIndex": "0x0",
        "to": "hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
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

#### txbyhash
- Description
Get transaction information by transaction hash
- Usage
```bash
usage: tbears txbyhash [-h] [-u URI] [-c CONFIG] hash

Get transaction by transaction hash

positional arguments:
  hash                  Hash of the transaction to be queried.

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
# update SCORE
$ tbears txbyhash 0xa2e1941dd3aaf607d05aecd48af6eb8108be32bc3368fe30ea58d7dfad907551
Transaction: {
    "jsonrpc": "2.0",
    "result": {
        "version": "0x3",
        "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
        "value": "0x0",
        "stepLimit": "0x10000000",
        "timestamp": "0x5815832d55c63",
        "nid": "0x3",
        "nonce": "0x1",
        "to": "cxd5dd40e1565bc97910ba97a40907d43574a05aa4",
        "data": {
            "contentType": "application/zip",
            "content": "0x504b03041400000008000655484e0000000002000000000000001800000070726f6a6563745f746573742f5f5f696e69745f5f2e70790300504b03041400000008000655484e77ae7f01420000005a0000001900000070726f6a6563745f746573742f7061636b6167652e6a736f6eabe6520002a5b2d4a2e2ccfc3c252b0525033d033d43251d88786e62665e7c5a664e2a48a6a0283f2b35b924be24b5b804454171727e11584508502618cce1aa0500504b03041400000008000655484e78e4e808dc000000a20100001c00000070726f6a6563745f746573742f70726f6a6563745f746573742e70798d904d4bc4301040eff915e35e9a48dd1f50587145d815c48bbd97b499d640365326891fffde3648aa17710e214cde7b878c4c17b003f980fc6607047b9989235c0bd11e4f7080aac5105f0662ac84189c0e01ca463e2e62beddeb80aa110296313842d7596f63d7c9806eacc1f40d14f64147ddaf3cdcdcc233796cb2b64e4833b254fba29b5e6d51f2cb3a44ed5ccefee1ff207ffb69363ae23ff46f50e5b77cdce14744f6da49466dc8bbcf43cb0955c9bfa273b4a543e4adfc44d384bc37d8a7498ed579456b782776e6aaaa61f96a5558c698d8c32e433bf105504b010214031400000008000655484e000000000200000000000000180000000000000000000000a4810000000070726f6a6563745f746573742f5f5f696e69745f5f2e7079504b010214031400000008000655484e77ae7f01420000005a000000190000000000000000000000a4813800000070726f6a6563745f746573742f7061636b6167652e6a736f6e504b010214031400000008000655484e78e4e808dc000000a20100001c0000000000000000000000a481b100000070726f6a6563745f746573742f70726f6a6563745f746573742e7079504b05060000000003000300d7000000c70100000000",
            "params": {}
        },
        "dataType": "deploy",
        "signature": "meNJOxl5yFwY54Qks9vNAP7AgbNF78uRIPYWEUCfKrxZy5x0Cm5OS7bekdevWQY0tym2ENnMiggzZyF/cdemHwA=",
        "txHash": "0xa2e1941dd3aaf607d05aecd48af6eb8108be32bc3368fe30ea58d7dfad907551",
        "txIndex": "0x0",
        "blockHeight": "0x10",
        "blockHash": "0xb8f13a792808f8be5a7169d51f5562ab9913aaccb925d048cb6e4877d0659041"
    },
    "id": 1
}
# transfer ICX
$ tbears txbyhash 0x6334856b5da9b2d4d841c1bbfa27fc0a0f3568d17bb9cac91302ce64f83aa6db
Transaction: {
    "jsonrpc": "2.0",
    "result": {
        "version": "0x3",
        "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
        "value": "0xde0b6b3a7640000",
        "stepLimit": "0xf4240",
        "timestamp": "0x58158a99c47be",
        "nid": "0x3",
        "nonce": "0x1",
        "to": "hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "signature": "p+e2Ko4QmJtBrTupfeAIcHNndAmQuXyNMPPS+1VnWc01CYKMcQATerEAFLTWnjSVvrloYhCiDiyclZg3wpqDxwA=",
        "txHash": "0x6334856b5da9b2d4d841c1bbfa27fc0a0f3568d17bb9cac91302ce64f83aa6db",
        "txIndex": "0x0",
        "blockHeight": "0xd7",
        "blockHash": "0x59ec3abfada762374c9cfd058d1d950f5e22c989e01bcb9b15c378d980ed83aa"
    },
    "id": 1
}
```

#### lastblock
- Description
Query last block information. When running on T-Bears service, "merkle_tree_root_hash" and "signature" will be empty.
- Usage
```bash
usage: tbears lastblock [-h] [-u URI] [-c CONFIG]

Get last block's info

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ tbears lastblock
block info : {
    "jsonrpc": "2.0",
    "result": {
        "version": "tbears",
        "prev_block_hash": "815c0fd7a0dd4594bb19ee39030c1bd91c200878f1f456fe8dd7ff4e0a19b839",
        "merkle_tree_root_hash": "tbears_block_manager_does_not_support_block_merkle_tree",
        "time_stamp": 1533719896011654,
        "confirmed_transaction_list": [
            {
                "version": "0x3",
                "from": "hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
                "value": "0x0",
                "stepLimit": "0x3000000",
                "timestamp": "0x572e8fd95db26",
                "nid": "0x3",
                "nonce": "0x1",
                "to": "cx0000000000000000000000000000000000000000",
                "data": {
                    "contentType": "application/zip",
                    "content": "0x32b34cfa39993fa093e",
                    "params": {}
                },
                "dataType": "deploy",
                "signature": "sig"
            }
        ],
        "block_hash": "28e6e4710c56e053920b95df0058317a4ac641b16d17d64db7f958e8a5650391",
        "height": 322,
        "peer_id": "fb5f43dc-9aeb-11e8-a31b-acde48001122",
        "signature": "tbears_block_manager_does_not_support_block_signature"
    },
    "id": 1
}
```

#### blockbyheight
- Description
Get block information using given block height.
- Usage
```bash
usage: tbears blockbyheight [-h] [-u URI] [-c CONFIG] height

Get block info using given block height

positional arguments:
  height                height of the block to be queried.

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ tbears blockbyheight 0
block info : {
    "jsonrpc": "2.0",
    "result": {
        "version": "tbears",
        "prev_block_hash": "",
        "merkle_tree_root_hash": "tbears_block_manager_does_not_support_block_merkle_tree",
        "time_stamp": 1549590537907591,
        "confirmed_transaction_list": [
            {
                "nid": "0x3",
                "accounts": [
                    {
                        "name": "genesis",
                        "address": "hx0000000000000000000000000000000000000000",
                        "balance": "0x2961fff8ca4a62327800000"
                    },
                    {
                        "name": "fee_treasury",
                        "address": "hx1000000000000000000000000000000000000000",
                        "balance": "0x0"
                    },
                    {
                        "name": "test1",
                        "address": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
                        "balance": "0x2961fff8ca4a62327800000"
                    }
                ]
            }
        ],
        "block_hash": "ecde4cc74881d75c65a3009f68c6ed00e7eb247fffb31c6317ba105cbec53fba",
        "height": 0,
        "peer_id": "",
        "signature": ""
    },
    "id": 1
}
```

#### blockbyhash
- Description
Get block information using given block hash.
- Usage
```bash
usage: tbears blockbyhash [-h] [-u URI] [-c CONFIG] hash

Get block info using given block hash

positional arguments:
  hash                  Hash of the block to be queried.

optional arguments:
  -h, --help            show this help message and exit
  -u URI, --node-uri URI
                        URI of node (default: http://127.0.0.1:9000/api/v3)
  -c CONFIG, --config CONFIG
                        Configuration file path. This file defines the default
                        value for the "uri" (default:
                        ./tbears_cli_config.json)
```
- Example
```bash
$ tbears blockbyhash 0x59ec3abfada762374c9cfd058d1d950f5e22c989e01bcb9b15c378d980ed83aa
block info : {
    "jsonrpc": "2.0",
    "result": {
        "version": "tbears",
        "prev_block_hash": "bb808a8596a05da22b629d94a0b9b5b33210c1b2c8ea9651b03160c34ba6b82e",
        "merkle_tree_root_hash": "tbears_block_manager_does_not_support_block_merkle_tree",
        "time_stamp": 1549592688479632,
        "confirmed_transaction_list": [
            {
                "version": "0x3",
                "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
                "value": "0xde0b6b3a7640000",
                "stepLimit": "0xf4240",
                "timestamp": "0x58158a99c47be",
                "nid": "0x3",
                "nonce": "0x1",
                "to": "hxaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
                "signature": "p+e2Ko4QmJtBrTupfeAIcHNndAmQuXyNMPPS+1VnWc01CYKMcQATerEAFLTWnjSVvrloYhCiDiyclZg3wpqDxwA=",
                "txHash": "0x6334856b5da9b2d4d841c1bbfa27fc0a0f3568d17bb9cac91302ce64f83aa6db"
            }
        ],
        "block_hash": "59ec3abfada762374c9cfd058d1d950f5e22c989e01bcb9b15c378d980ed83aa",
        "height": 215,
        "peer_id": "b313cb28-2b43-11e9-93fc-acde48001122",
        "signature": "tbears_block_manager_does_not_support_block_signature"
    },
    "id": 1
}
```

### T-Bears Utility Commands
Commands that generate configuration file and keystore file.
#### keystore
- Description
Create a keystore file in the given path. Generate a private and public key pair.
- Usage
```bash
usage: tbears keystore [-h] [-p PASSWORD] path

Create keystore file in the specified path. Generate privatekey, publickey
pair using secp256k1 library.

positional arguments:
  path                  Path of keystore file.

optional arguments:
  -h, --help            show this help message and exit
  -p PASSWORD, --password PASSWORD
                        Keystore file's password
```
- Example
```bash
$ tbears keystore -p short ./keystore_example
Password must be at least 8 characters long including alphabet, number, and special character.
$ tbears keystore -p p@ssw0rd ./keystore_example
Made keystore file successfully
```

#### genconf
- Description
Generate T-Bears default config files. 
`tbears_cli_config.json` : config file for T-Bears CLI
`tbears_server_config.json` : config file for T-Bears Service
`keystore_test1` : keystore file for `test1` account
- Usage
```bash
usage: tbears genconf [-h]

Generate tbears config files. (tbears_server_config.json,
tbears_cli_config.json and keystore_test1)

optional arguments:
  -h, --help  show this help message and exit
```
- Example
```bash
$ tbears genconf
Made tbears_cli_config.json, tbears_server_config.json, keystore_test1 successfully
$ tbears genconf
There were configuration files already.
```

#### console
- Description
Enter T-Bears interactive mode using IPython. ([Ipython.org](https://ipython.org/))
- Usage
```bash
usage: tbears console [-h]

Get into tbears interactive mode by embedding IPython

optional arguments:
  -h, --help  show this help message and exit
```
- Example
In the interactive mode, you can execute command in short form (without `tbears`) by predefined IPython's magic command.
TAB will complete T-Bears's command or variable names. Use TAB.
```bash
$ tbears console
Python 3.6.5 (v3.6.5:f59c0932b4, Mar 28 2018, 03:03:55)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.4.0 -- An enhanced Interactive Python. Type '?' for help.

IPython profile: tbears

tbears) start
Started tbears service successfully

tbears) balance hxef73db5d0ad02eb1fadb37d0041be96bfa56d4e6
balance in hex: 0x0
balance in decimal: 0
```
You can access nth-output using `_nth` expression.
The `Out` command displays the index and its outputs in a dictionary format.
```bash
tbears) '1'
'1'

tbears) 'second'
'second'

tbears) 3
3

tbears) _2
'second'

tbears) Out
{1: '1', 2: 'second', 3: 3, 4: 'second'}
```
You can pass the value of a variable as an argument by prefixing the variable name with "$".
```bash
tbears) address = f"hx{'0'*40}"

tbears) balance $address
balance in hex: 0x2961fff8ca4a62327800000
balance in decimal: 800460000000000000000000000
```
In the interactive mode, `deployresults` command is available to list up the SCOREs that have been deployed while T-Bears interactive mode is running.
```bash
tbears) deployresults
1.path : abc/, txhash : 0x583a89ec656d71d1641945a39792e016eefd6221ad536f9c312957f0c4336774, deployed in : http://127.0.0.1:9000/api/v3
2.path : token/, txhash : 0x8c2fe3c877d46b7a1ba7feb117d0b12c8b88f33517ad2315ec45e8b7223c22f8, deployed in : http://127.0.0.1:9000/api/v3
3.path : abctoken/, txhash : 0xee6e311d2652fd5ed5981f4906bca5d4d6933400721fcbf3528249d7bf460e42, deployed in : http://127.0.0.1:9000/api/v3

```
T-Bears assigns T-Bears command execution result to `_r` variable.
You should use "{}" expression when you pass a member of list or dictionary.
```bash
tbears) deploy sample_token
Send deploy request successfully.
transaction hash: 0x5257b44fe0f36c492e255dbfcdb2ca1134dc9a942b875241d01db3d36ac2bdc8

tbears) result = _r

tbears) result
{'jsonrpc': '2.0',
 'result': '0x5257b44fe0f36c492e255dbfcdb2ca1134dc9a942b875241d01db3d36ac2bdc8',
 'id': 1}

tbears) txresult {result['result']}
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0x3f4cf436d6f10eb04c7b9e805f497053652227ecc6f8e95bad8941f1eff3b01b",
        "blockHeight": "0x1dc",
        "blockHash": "0xdd7e497a3fd4f5c759a06a8f7a284b1be149bf871f1dd370ff84e65d0865c37f",
        "txIndex": "0x0",
        "to": "cx0000000000000000000000000000000000000000",
        "scoreAddress": "cx97fc69e1c7ea56adf009133af460c0a436727960",
        "stepUsed": "0x1513d98",
        "stepPrice": "0x0",
        "cumulativeStepUsed": "0x1513d98",
        "eventLogs": [],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 3
}

tbears) scoreapi {_r['result']['scoreAddress']}
SCORE API: [
    {
        "type": "fallback",
        "name": "fallback",
        "inputs": []
    },
    {
        "type": "function",
        "name": "hello",
        "inputs": [],
        "outputs": [
            {
                "type": "str"
            }
        ],
        "readonly": "0x1"
    }
]

```

## References
- [ICON JSON-RPC API v3](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md)
