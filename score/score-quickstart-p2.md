---
title: "Part 2. HelloWorld on testnet"
excerpt: ""
---

In this part 2, we will deploy the HelloWorld SCORE onto the testnet. You will learn how to configure your tools to interact with the testnet.

## Configure ICONex to connect to the testnet

If you have not installed ICON Chrome wallet, here is the chrome web store [link](https://chrome.google.com/webstore/detail/iconex/flpiciilemghbmfalicajoolhkkenfel).

ICON Chrome wallet is connected to the mainnet. You can switch to other networks by changing its configuration. 

- Open the Chrome DevTools by pressing F12, then go to the **Application** tab. In the **Storage** section, expand **Local Storage**.
- Add a new key/value pair, **isDev/true**, by clicking on the empty row at the bottom of the table.
- Reload your wallet, then you will see the menu in the bottom. Click the **ICX (SERVER)** button to open the dropup list of the available networks. Select **YEOUIDO** network, this is a testnet for DApp developers. 

Please refer to the [network guide](the-icon-network) for more information.



## Create an account in ICONex

Let's create an account in ICONex, and download the keystore file. Full guideline of creating a keyfile is [here](account-management).

We will use this keystore file on testnet. The keystore file downloaded from ICONex will look something like `UTC--2018-10-06T06_00_02.195Z--hxbac99ffea54749ca1c86ab4e6bfe0b630bf7a7a0`. As you may have noticed, the latter part of the filename is your address, `hxbac99ffea54749ca1c86ab4e6bfe0b630bf7a7a0` in this example. Let's rename the file to `keystore_test2` for human readability. 



## Get test ICX

You need ICX to deploy and invoke a SCORE on testnet, because every transaction on testnet requires a fee. To receive testnet ICX, please send email to `testicx@icon.foundation` with the following information.

- Testnet node url
- Address to receive the testnet ICX, the one you created and downloaded above.

Check your balance from CLI, or in the ICONex. 

```console
# tbears balance [account] -u https://bicon.net.solidwallet.io/api/v3
```



## Transaction fees

[Transaction fees explained](transaction-fees).  

- `stepLimit` in transaction request message

  Every transaction request message has a `stepLimit` field. This value can not exceed the "maximum step limit" defined in the Governance SCORE. 

  ```
  {
      "jsonrpc": "2.0",
      "method": "icx_sendTransaction",
      "id": 1234,
      "params": {
          "version": "0x3",
          "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
          "to": "cx9a4c4229ab2cbd61a5cc051fbbb6ee7e3e3adfac",
          "stepLimit": "0x123450",
           ...
  ```

- `stepUsed` in transaction result 

  You may have already noticed that every transaction result returns a `stepUsed` field. This is the actual amount of step consumed by the transaction. It is a good practice to test against T-Bears to figure out the step amount required by each transaction. On T-Bears, step is calculated but the step price is set to zero, so the transaction fee is always zero. T-Bears will return `Out of step` error only if the step usage reaches the `stepLimit` value in your transaction request. 

  ```
  {
      "jsonrpc": "2.0",
      "result": {
          "txHash": "0xc40cbbf2b89cd1e2890....",
          "blockHeight": "0x3158",
          "blockHash": "0x9e0c1385128bf0d4257....",
           ...
          "stepUsed": "0x4d361d0",
           ...
  ```

- `Out of step` error in transaction result

  When you receive an `Out of step` error, increase the `stepLimit` in the request message. The value should be larger than `stepUsed`.

  ```
  {
      "jsonrpc": "2.0", 
      "result": {
          "txHash": "0x9f512fe4b431d8780d75c29ad307f3....", 
          ...
          "stepUsed": "0x4d361d0",
          "status": "0x0", 
          "failure": {
              "code": "0x7d64", 
              "message": "Out of step: contractSet"
          }
          ...
  ```



## Configure T-Bears CLI to see the testnet

We will modify the default cli configuration file, "tbears_cli_config.json", for testnet. In this config file, we will set `uri`, `nid`, and `stepLimit`. 
Every CLI commands lookup the `uri` from the config file if `-u` option is not given. `deploy` command uses `nid` and `stepLimit` values in this file. 

```
{
    "uri": "https://bicon.net.solidwallet.io/api/v3",
    "nid": "0x3",
    "keyStore": null,
    "from": "hxe7af5fcfd8dfc67530a01a0e403882687528dfcb",
    "to": "cx0000000000000000000000000000000000000000",
    "stepLimit": "0x5000000",
    ...
}
```



## Deploy HelloWorld to the testnet (T-Bears CLI)

Unlike T-Bears emulated environment, on testnet, a valid signature is required to send a transaction. So, every transaction request will require a keystore file to sign it. 

`tbears deploy` command with `-k [keystore_file]` option will install the SCORE on testnet. Target network info (uri, nid) is read from the default config file. Don't forget to get the SCORE address from the transaction result. 

```console
root@07dfee84208e:/tbears# tbears deploy hello_world -k keystore_test2
root@07dfee84208e:/tbears# tbears txresult [txhash]
```



## Execute HelloWorld (T-Bears CLI, Python)

#### T-Bears CLI

Same command will invoke `hello` method on testnet. Read-only function call does not require keystore file. Testnet `uri` is read from the default config file. Just make sure you correctly updated `to` and `from` values in the "call.json" request message.

```console
root@07dfee84208e:/tbears# tbears call call.json

root@07dfee84208e:/tbears# cat call.json 
{
    "jsonrpc": "2.0",
    "method": "icx_call",
    "id": 1,
    "params": {
        "from": "hxbac99ffea54749ca1c86ab4e6bfe0b630bf7a7a0",  <-- test2 address
        "to": "cx9a4c4229ab2cbd61a5cc051fbbb6ee7e3e3adfac",  <-- SCORE address on testnet 
        "dataType": "call", 
        "data": {
            "method": "hello" 
        }
    }
}
```


#### Python SDK

We will create a "hello.py" to invoke a `hello` method of the contract on testnet. Use the actual SCORE address and keystore password in your code.

```python
from iconsdk.icon_service import IconService
from iconsdk.providers.http_provider import HTTPProvider
from iconsdk.wallet.wallet import KeyWallet
from iconsdk.builder.call_builder import CallBuilder

node_uri = "https://bicon.net.solidwallet.io/api/v3"
network_id = 3
hello_world_address = "__SCORE_ADDRESS__"
keystore_path = "./keystore_test2"
keystore_pw = "__PASSWORD__"

wallet = KeyWallet.load(keystore_path, keystore_pw)
tester_addr = wallet.get_address()
icon_service = IconService(HTTPProvider(node_uri))

call = CallBuilder().from_(tester_addr)\
                    .to(hello_world_address)\
                    .method("hello")\
                    .build()

result = icon_service.call(call)
print(result)
```

Because you don't need to sign your read-only function call request, creating a wallet instance from the keystore file is not a mandatory step. Wallet instance is created just to get your account address. 

Run the code. 

```console
$ python3 hello.py
Hello, hxbac99ffea54749ca1c86ab4e6bfe0b630bf7a7a0. My name is HelloWorld
```

