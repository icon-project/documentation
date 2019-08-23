---
title: "How to setup governance variables"
excerpt: "The source code is found on GitHub at https://github.com/icon-project/preptools"
---

This is a guideline for setting up governance variables using preptools.


## Intended Audience
We recommend all P-Rep candidates to go through this guideline and participate in block generation testing.

## Pre-requisites
### Requirements
* OS: MacOS, Linux
  * Windows is not supported yet.
* Python
  * Python 3.6.5+ or 3.7.x. **In the future, we will only support 3.7.x. ** Please upgrade python version to 3.7.x.**
  * Optional) We recommend creating an isolated Python 3 virtual environment with [virtualenv](https://virtualenv.pypa.io/en/stable/).
```bash
$ python -m venv venv             # Create a virtual environment.
$ source venv/bin/activate        # Enter the virtual environment.
```

### Installation
Github source for installation, https://github.com/icon-project/preptools
```bash
(venv) $ pip install git+https://github.com/icon-project/preptools.git
```

## How to setup Governance Variables
To Setup Governance Variables on Mainnet, you will need to specify nid, url and keystore, using cmdline or configuration.

* Using command line
  * Specify nid and url by command line.
```bash
(venv) $ preptools [command] -n 1 -u https://ctz.solidwallet.io/api/v3 -k keystore ...
```
* Using configuration
  * Specify nid and url by configuration.
```
(venv) $ cat preptools_config.json

{
    "url": "https://ctz.solidwallet.io/api/v3",
    "nid": 1,
    "keystore": keystore
}

(venv) $ preptools [command] ...
```


**Usage**
```bash
usage: preptools setGovernanceVariables [-h] [--url URL] [--nid NID]
                                        [--config CONFIG]
                                        [--password PASSWORD]
                                        [--keystore KEYSTORE] --irep [IREP]

optional arguments:
  -h, --help            show this help message and exit
  --url URL, -u URL     node url default(http://127.0.0.1:9000/api/v3)
  --nid NID, -n NID     networkId default(3) ex) mainnet(1), testnet(2)
  --config CONFIG, -c CONFIG
                        preptools config file path
  --password PASSWORD, -p PASSWORD
                        keystore password
  --keystore KEYSTORE, -k KEYSTORE
                        keystore file path
  --irep [IREP]         amounts of irep in loop
```

**Examples**

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

**Document**  
Further information can be found in the document below:
* [Setup Governance Variables](https://github.com/icon-project/preptools#preptools-setgovernancevariables)


## License
This project follows the Apache 2.0 License. Please refer to [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) for details.

