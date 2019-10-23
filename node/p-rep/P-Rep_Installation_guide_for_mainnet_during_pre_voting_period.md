This document is a guideline detailing how to install and operate a Public Representative (“P-Rep”) node on the MainNet using a docker.
P-Reps are the consensus nodes that produce, verify blocks and participate in network policy decisions on the ICON Network.
​
​
## Intended Audience

We recommend all P-Rep candidates to go through this guideline.
This is a temporary guideline for Pre-Voting.
​
​
## Pre-requisites

We assume that you have previous knowledge and experience in:
- IT infrastructure management
- Linux or UNIX system administration
- Docker container
- Linux server and docker service troubleshooting
​

## Requirements

- OS: MacOS, Linux
Windows is not supported yet.
- Docker, docker-compose
​

## How to install prep-node for mainnet

Citizen node is used for block sync and API load balancing without network consensus.
Non-P-Rep nodes act in citizen node and synchronize blocks.
​
P-Reps can participate either by installing a citizen node or connect to `https://ctz.solidwallet.io`.
​
In order to facilitate block sync, we provide a snapshot which gets activated when  "FASTEST_START" is "yes". Then, snapshot in "data volume" will be re-downloaded when the null size file is deleted.

If the snapshot file already exists, a new file will not be downloaded.
​
docker and docker-compose need to be installed beforehand.

Citizen-node must be run with the wallet(keystore file) that registered as prep.

The `keystore file` needs to be exported and stored in the `cert` directory
Below is a directory structure under the docker-compose.yml

```yaml

|-- docker-compose.yml   
  |-- data  → data directory            
       |-- mainnet  → block DB directory 
       |-- log  → log directory
  |-- cert  → keytore or cert key directory
       |-- YOUR_KEYSTORE_FILE  → put your keystore file

```

(How to export keystore file → https://www.icondev.io/docs/account-management)  

​
**Using docker-compose command (Recommended)**
​
Open 'docker-compose.yml' in a text editor and add the following content:
```yaml
version: "3"
services:
  prep-node:
     image: "iconloop/prep-node:1910211829xc2286d"
     container_name: "prep-mainnet"
     network_mode: host     
     restart: "always"
     environment:
        NETWORK_ENV: "mainnet"
        LOG_OUTPUT_TYPE: "file"
        SWITCH_BH_VERSION3: "10324749"
        CERT_PATH: "/cert"
        LOOPCHAIN_LOG_LEVEL: "DEBUG"
        ICON_LOG_LEVEL: "DEBUG"        
        FASTEST_START: "yes" # Restore from lastest snapshot DB
        PRIVATE_KEY_FILENAME: "{YOUR_KEYSTORE or YOUR_CERTKEY FILENAME}" # only file name
        PRIVATE_PASSWORD: "{YOUR_KEY_PASSWORD}"
     cap_add:
        - SYS_TIME      
     volumes:
        - ./data:/data # mount a data volumes
        - ./cert:/cert # Automatically generate cert key files here
     ports:
        - 9000:9000
        - 7100:7100
```

**Run docker-compose**

```bash
$ docker-compose up -d
```
​
​
```conf
prep-node_1  |   [2019-10-23 15:56:21.101] Your IP: 58.234.156.140
prep-node_1  |   [2019-10-23 15:56:21.106] RPC_PORT: 9000 / RPC_WORKER: 3
prep-node_1  |   [2019-10-23 15:56:21.110] DEFAULT_PATH=/data/mainnet in Docker Container
prep-node_1  |   [2019-10-23 15:56:21.115] DEFAULT_LOG_PATH=/data/mainnet/log
prep-node_1  |   [2019-10-23 15:56:21.120] DEFAULT_STORAGE_PATH=/data/mainnet/.storage
prep-node_1  |   [2019-10-23 15:56:21.124] scoreRootPath=/data/mainnet/.score_data/score
prep-node_1  |   [2019-10-23 15:56:21.129] stateDbRootPath=/data/mainnet/.score_data/db
prep-node_1  |   [2019-10-23 15:56:21.133] Time synchronization with NTP / NTP SERVER: time.google.com
prep-node_1  |    23 Oct 15:56:27 ntpdate[35]: adjust time server 216.239.35.0 offset -0.000066 sec
prep-node_1  |   [2019-10-23 15:56:27.912] P-REP package version info - _1910211829xc2286d
prep-node_1  |   [2019-10-23 15:56:28.394] iconcommons             1.1.2
prep-node_1  |                             iconrpcserver           1.4.5
prep-node_1  |                             iconsdk                 1.2.0
prep-node_1  |                             iconservice             1.5.15
prep-node_1  |                              loopchain               2.4.15
prep-node_1  |   [2019-10-23 15:56:28.399] SERVICE_API = https://ctz.solidwallet.io/api/v3 / SUBSCRIBE_USE_HTTPS=true
prep-node_1  |   [2019-10-23 15:56:28.404] NETWORK_ENV=mainnet, SERVICE=mainnet, ENDPOINT_URL=https://ctz.solidwallet.io
prep-node_1  |   [2019-10-23 15:56:28.419] Already cert keys= /cert/prep_test1.key
prep-node_1  |   [2019-10-23 15:56:29.085] UnRegistered P-Rep Network
prep-node_1  |   [2019-10-23 15:56:29.251] CHANNEL_MANAGE_DATA not found - /prep_peer/conf/channel_manange_data.json
prep-node_1  |   [2019-10-23 15:56:29.344] START FASTEST MODE : NETWORK_NAME=MainctzNet
prep-node_1  |   [2019-10-23 15:56:29.486] Start download
```
