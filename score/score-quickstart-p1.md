---
title: "Part 1. HelloWorld on local emulated environment"
excerpt: ""
---

## Install T-Bears (Docker)

- Install Docker [[Get Started with Docker](https://www.docker.com/get-started)]

- Install T-Bears and run the container.

Below command will download tbears docker image, create a container, start the container, and attach your stdin/stderr to the container. 

```console
$ docker run -it --name local-tbears -p 9000:9000 iconloop/tbears
 * Starting RabbitMQ Messaging Server rabbitmq-server                    [ OK ]
Made tbears_cli_config.json, tbears_server_config.json, keystore_test1 successfully
Started tbears service successfully
root@c5b81f9874ee:/tbears#
```

Exit and stop the container.

```console
root@07dfee84208e:/tbears# exit
```

Show the container list.

```console
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS                    NAMES
c5b81f9874ee        iconloop/tbears     "entry.sh"          6 minutes ago       Up 6 minutes               0.0.0.0:9000->9000/tcp   local-tbears
```

Start and attach to the container, or attach to a running container.

```console
$ docker container start -a local-tbears
root@07dfee84208e:/tbears#

$ docker container attach local-tbears
root@07dfee84208e:/tbears#
```



## Test account

Test keystore file, `keystore_test1`, is provided for your convenience. Be careful not to use this keystore file  to sign your transaction to any network other than T-Bears, because the password is open to public. 

- Address : hxe7af5fcfd8dfc67530a01a0e403882687528dfcb
- Password : test1_Account
- ICX balance : 0x2961fff8ca4a62327800000



## Check your account balance

Let's issue a `tbears balance` command to check your balance. 

```console
root@07dfee84208e:/tbears# tbears balance hxe7af5fcfd8dfc67530a01a0e403882687528dfcb
balance in hex: 0x2961fff8ca4a62327800000
balance in decimal: 800460000000000000000000000
```



## Create HelloWorld contract and deploy it

`tbears init` command will initialize a project. 

```console
root@07dfee84208e:/tbears# tbears init hello_world HelloWorld
Initialized tbears successfully

root@07dfee84208e:/tbears# ls hello_world
hello_world.py  __init__.py  package.json

```

`hello_world.py` has the SCORE implementation template which is fully functional.

```python
from iconservice import *

class HelloWorld(IconScoreBase):

    def __init__(self, db: IconScoreDatabase) -> None:
        super().__init__(db)

    def on_install(self) -> None:
        super().on_install()

    def on_update(self) -> None:
        super().on_update()

    @external(readonly=True)
    def hello(self) -> str:
        return "Hello"
```

Let's deploy the contract without modification and get the contract address. For every transaction, you need to check the execution result using txhash. Transaction result contains the SCORE address if the deployment was successful. 

```console
root@07dfee84208e:/tbears# tbears deploy hello_world
Send deploy request successfully.
If you want to check SCORE deployed successfully, execute txresult command
transaction hash: 0xc40cbbf2b89cd1e2890132145e6d86ad61835edaca0bcc3a4c34b5cb22b8be28

root@07dfee84208e:/tbears# tbears txresult 0xc40cbbf2b89cd1e2890132145e6d86ad61835edaca0bcc3a4c34b5cb22b8be28
Transaction result: {
    "jsonrpc": "2.0",
    "result": {
        "txHash": "0xc40cbbf2b89cd1e2890132145e6d86ad61835edaca0bcc3a4c34b5cb22b8be28",
        "blockHeight": "0x3158",
        "blockHash": "0x9e0c1385128bf0d425773f9f9130d683d327a058a9c8dc0a6c4df71bb98195e1",
        "txIndex": "0x0",
        "to": "cx3176b5d6cae66a1abbc3ca9070423a5c708834a9",
        "scoreAddress": "cx3176b5d6cae66a1abbc3ca9070423a5c708834a9", <-- SCORE address
        "stepUsed": "0x4d361d0",
        "stepPrice": "0x0",
        "cumulativeStepUsed": "0x4d361d0",
        "eventLogs": [],
        "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
        "status": "0x1"
    },
    "id": 1
}
```

SCORE has been successfully deployed. We will invoke `hello` method from CLI and see the result. `call.json` file contains the request message. `to` is the SCORE address, `from` is the message sender's address. Don't forget to provide the actual SCORE address, the one you got after deploy. For the complete json message format, please refer to [ICON JSON-RPC API v3 specification](https://github.com/icon-project/icon-rpc-server/blob/master/docs/icon-json-rpc-v3.md).

```console
root@07dfee84208e:/tbears# tbears call call.json
response : {
    "jsonrpc": "2.0",
    "result": "Hello",
    "id": 1
}
root@07dfee84208e:/tbears# cat call.json 
{
    "jsonrpc": "2.0",
    "method": "icx_call",
    "id": 1,
    "params": {
        "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
        "to": "cx3176b5d6cae66a1abbc3ca9070423a5c708834a9",
        "dataType": "call", 
        "data": {
            "method": "hello" 
        }
    }
}
```



## Modify HelloWorld contract to greet you

It would be a pleasure to have a more agreeable HelloWorld SCORE. Let's give it a name, and greet you with a nicer way. One method will be added, `hello` method will be modified.

- `name` is a read-only funtion, and returns the SCORE name.

- `hello` method is slightly modified to recognize you. `msg` is a built-in property that holds information of the message sender, and the amount of ICX the sender attempts to transfer. They can be referenced by `msg.sender` and `msg.value` respectively. 

```python
    @external(readonly=True)
    def name(self) -> str:
        return "HelloWorld"
    
    @external(readonly=True)
    def hello(self) -> str:
        return f'Hello, {self.msg.sender}. My name is {self.name()}'
```

We need to deploy the SCORE again to reflect the change. `deploy` command with  `-m update` and `-o [score_address]` options will update the SCORE. Updating is a unique feature of ICON. SCORE address is not changed after update.

```console
root@07dfee84208e:/tbears# tbears deploy -m update -o cx3176b5d6cae66a1abbc3ca9070423a5c708834a9 hello_world
Send deploy request successfully.
If you want to check SCORE deployed successfully, execute txresult command
transaction hash: 0xc412dc9c6685701c8837eddea091283244303d322aa1fba36bc0782e1b483763
...
```

Invoke the `hello` method with the same request message, and see the changes. 

```console
root@07dfee84208e:/tbears# tbears call call.json
response : {
    "jsonrpc": "2.0",
    "result": "Hello, hxe7af5fcfd8dfc67530a01a0e403882687528dfcb. My name is HelloWorld",
    "id": 1
}
```

