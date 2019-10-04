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

## How to install citizen node

Citizen node is used for block sync and API load balancing without network consensus.
Non-P-Rep nodes act in citizen node and synchronize blocks.
​
P-Reps can participate either by installing a citizen node or connect to `https://ctz.solidwallet.io`.
​
In order to facilitate block sync, we provide a snapshot which gets activated when  "FASTEST_START" is "yes". Then, snapshot in "data volume" will be re-downloaded when the null size file is deleted.

If the snapshot file already exists, a new file will not be downloaded.
​
docker and docker-compose need to be installed beforehand.
​
**Using docker-compose command (Recommended)**
​
Open 'docker-compose.yml' in a text editor and add the following content:
```yaml
version: "3"
services:
  citizen:
     image: "iconloop/citizen-node:1908271151xd2b7a4"
     network_mode: host
     environment:
        LOG_OUTPUT_TYPE: "file"
        LOOPCHAIN_LOG_LEVEL: "DEBUG"
        FASTEST_START: "yes" # Restore from lastest snapshot DB
      
     volumes:
        - ./data:/data # mount a data volumes
        - ./keys:/citizen_pack/keys # Automatically generate cert key files here
     ports:
        - 9000:9000
```

**Run docker-compose**

```bash
$ docker-compose up -d
```
​
​
```conf
citizen_1 | [2019-08-20 15:36:18.933] Your IP: 58.234.156.140
citizen_1 | [2019-08-20 15:36:18.935] RPC_PORT: 9000 / RPC_WORKER: 3
citizen_1 | [2019-08-20 15:36:18.937] DEFAULT_STORAGE_PATH=/data/loopchain/mainnet/.storage in Docker Container
citizen_1 | [2019-08-20 15:36:18.939] scoreRootPath=/data/loopchain/mainnet/.score_data/score
citizen_1 | [2019-08-20 15:36:18.940] stateDbRootPath=/data/loopchain/mainnet/.score_data/db
citizen_1 | [2019-08-20 15:36:18.942] Citizen package version info - 1908271151xd2b7a4
citizen_1 | WARNING: You are using pip version 19.1.1, however version 19.2.2 is available.
citizen_1 | You should consider upgrading via the 'pip install --upgrade pip' command.
citizen_1 | iconcommons             1.1.2
citizen_1 | iconrpcserver           1.3.1.1
citizen_1 | iconservice             1.4.2
citizen_1 | loopchain               2.2.1.3
citizen_1 | [2019-08-20 15:36:19.448] builtinScoreOwner = hx677133298ed5319607a321a38169031a8867085c
citizen_1 | 0
citizen_1 | [2019-08-20 15:36:19.528] START FASTEST MODE : NETWORK_NAME=MainctzNet
citizen_1 | [2019-08-20 15:36:19.777] Start download - https://s3.ap-northeast-2.amazonaws.com/icon-leveldb-backup/MainctzNet/20190820/MainctzNet_BH7344686_data-20190820_1500.tar.gz
citizen_1 | [OK] CHECK=0, Download 20190820/MainctzNet_BH7344686_data-20190820_1500.tar.gz(/data/loopchain/mainnet/) to /data/loopchain/mainnet
```