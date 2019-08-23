---
title: "How to Register, Unregister and Set P-Rep"
excerpt: "The source code is found on GitHub at https://github.com/icon-project/preptools"
---

This document is a guideline detailing how to register, unregister, and set a Public Representative (“P-Rep”) node on the MainNet using preptools. 
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
Install from github source, https://github.com/icon-project/preptools
```bash
(venv) $ pip install git+https://github.com/icon-project/preptools.git
```

## How to Register, Unregister and Set P-Rep to MainNet
In order to register, unregister and set P-Rep to MainNet,  specify nid, url and keystore either using cmdline or configuration.

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
    "keystore": "/home/user/keystore"
} 
(venv) $ preptools [command] ...
```

### Register P-Rep
#### Usage
```bash
usage: preptools registerPRep [-h] [--url URL] [--nid NID] [--config CONFIG]
                              [--password PASSWORD] [--keystore KEYSTORE]
                              [--name NAME] [--country COUNTRY] [--city CITY]
                              [--email EMAIL] [--website WEBSITE]
                              [--details DETAILS] [--p2p-endpoint P2PENDPOINT]
                              [--prep-json PREP_JSON]
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
  --name NAME           P-Rep name
  --country COUNTRY     P-Rep's country
  --city CITY           P-Rep's city
  --email EMAIL         P-Rep's email
  --website WEBSITE     P-Rep's homepage url
  --details DETAILS     json url including P-Rep detailed information 
  --p2p-endpoint P2PENDPOINT
                        Network info used for connecting among P-Rep nodes
  --prep-json PREP_JSON
                        json file including P-Rep information
```

#### Example
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
(venv) $ preptools registerPRep --prep-json registerPRep.json -k keystore

# Using optional argument and interactive command line
(venv) $ preptools registerPRep -k keystore --name "kokoa node" --email "kokoa@example.com"
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
```

**Document**  
Further information can be found in the below document:
* [Register P-Rep using preptools](https://github.com/icon-project/preptools#preptools-registerprep)

### Unregister P-Rep
In order to unregister as a P-Rep, use P-Rep's keystore file.

```bash
usage: preptools unregisterPRep [-h] [--url URL] [--nid NID] [--config CONFIG]
                                [--password PASSWORD] [--keystore KEYSTORE]
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
# with keystore path
(venv) $ preptools unregisterPRep -k keystore
# when keystore path is in configure file
(venv) $ preptools unregisterPRep 
```

**Document**  
Further information can be found in the below document:
* [Unregister P-Rep using preptools](https://github.com/icon-project/preptools#preptools-unregisterprep)

### Set P-Rep
#### Usage
```bash
usage: preptools setPRep [-h] [--url URL] [--nid NID] [--config CONFIG]
                         [--password PASSWORD] [--keystore KEYSTORE] [-i]
                         [--name NAME] [--country COUNTRY] [--city CITY]
                         [--email EMAIL] [--website WEBSITE]
                         [--details DETAILS] [--p2p-endpoint P2PENDPOINT]
                         [--prep-json PREP_JSON]
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
  -i, --interactive     Activate interactive mode when prep fields are blank.
  --name NAME           PRep name
  --country COUNTRY     P-Rep's country
  --city CITY           P-Rep's city
  --email EMAIL         P-Rep's email
  --website WEBSITE     P-Rep's homepage url
  --details DETAILS     json url including P-Rep detailed information 
  --p2p-endpoint P2PENDPOINT
                        Network info used for connecting among P-Rep nodes
  --prep-json PREP_JSON
                        json file including P-Rep information
```

#### Example
Update P-Rep using json file, optional argument and interactive command line.

```bash
(venv) $ cat setPRep.json
{
    "name": "kokoa node",
    "country": "USA",
    "website": "https://icon.kokoa.com"
}

# with json
(venv) $ preptools setPRep --prep-json setPRep.json -k keystore

# with json and arguments
(venv) $ preptools setPRep --prep-json setPRep.json -k keystore --name "banana node" --email "example@iconloop.com" -i
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
(venv) $ preptools setPRep --prep-json setPRep.json -k keystore --name "banana node" --email "example@iconloop.com" -i
> Password: 
 > city : 
 > details : 
 > p2pEndpoint : 127.0.0.1:9000
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
        "p2pEndpoint": "127.0.0.1:9000"
    }
}
```

**Document**  
Further information can be found in the document below:
* [Set P-Rep using preptools](https://github.com/icon-project/preptools#preptools-setprep)

## JSON Standard for Public Representative Detailed Information 

This is the JSON standard for detailed information about the P-Rep. P-Rep can submit the url of detailed information via the `registerPRep` and `setPRep` action on the ICON Blockchain. We strongly recommend that you register this information.

```json
{
    "representative": {
        "logo": {
            "logo_256": "https://icon.foundation/img/img-256.png",
            "logo_1024": "https://icon.foundation/img/img-1024.png",
            "logo_svg": "https://icon.foundation/img/img-logo.svg"
        },
        "media": {
            "steemit": "",
            "twitter": "",
            "youtube": "",
            "facebook": "",
            "github": "",
            "reddit": "",
            "keybase": "",
            "telegram": "",
            "wechat": ""
        }
    },
    "server": {
        "location": {
            "country": "",
            "city": ""
        },
        "server_type": "",
        "api_endpoint": ""
    }
}
```

- representative: Basic information of Public Representative
  - logo: Logo images of P-Rep
    -  logo_256: image 256x256px
    -  logo_1024: image 1024x1024px
    -  logo_sgv: image svg
  - media: URL and username of social media
    -  steemit: Steemit URL
    -  twitter: Twitter URL
    -  youtube: Youtube URL
    -  Facebook: Facebook URL
    -  github: Github URL
    -  reddit: Raddit URL
    -  keybase: Username
    -  telegram: Username
    -  wechat: Username
- server: Server information of Public Representative
  - location: Server location
    -  name: Node location in human readable format [City, State]
    -  country: Node country code [XX]
  - server_type: Type of server ‘cloud, on-premise, hybrid’
  - api_endpoint: HTTP endpoint http://host:port

### How to use
Create a JSON file and upload it to your domain server. When you call the `registerPRep` or `setPRep` function, input the url of this file into the `details` field.

## License
This project follows the Apache 2.0 License. Please refer to [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) for details.
