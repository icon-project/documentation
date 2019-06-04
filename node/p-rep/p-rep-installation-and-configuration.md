---
title: "P-Rep Installation and Configuration"
excerpt: "General information about the ICON P-Rep election - https://icon.community/iconsensus/"
---

This document is a guideline about how to install and operate a Public Representative (“P-Rep”) node on the Testnet using a docker. P-Reps are the consensus nodes that produce, verify blocks and participate in network policy decisions on the ICON Network. The purpose of the Testnet is to provide a test environment for the P-Rep candidates. The Candidates can simulate various activities such as block production to check technical stability of the nodes on the Testnet. (The node operation guideline on the ICON Mainnet for the elected P-Reps will be released in the future.)


## Intended Audience

We recommend all P-Rep candidates to go through this guideline and participate in the block generation test.



## Pre-requisites

We assume that you have previous knowledge and experience in:

- IT infrastructure management
- Linux or UNIX system administration
- Docker container
- Linux server and docker service troubleshooting

### HW Requirements for Testnet

Below specification is a minimum requirement for the testnet application. 

| Desciption    | Minimum Specification for P-Rep TestNet Application | 
| ------------- | ---------------------------------------------| 
| CPU Model     | Intel(R) Xeon(R) CPU @ 3.00 GHz | 
| vCPU (core)   | 2                                            |
| RAM           | 4 G                                          |
| Disk          | 100 G                                | 
| Network       | 1 Gbps                                       | 


### SW Requirements

#### OS requirements

- Linux (CentOS 7 or Ubuntu 16.04+)

#### Package requirements

- Docker 18.x or higher

For your reference, ICON node depends on the following packages. The packages are included in the P-Rep docker image that we provide, so you don't need to install them separately. 

- Python 3.6.5 or higher (3.7 is not supported)
- RabbitMQ 3.7 or higher



## Network Diagram of P-Rep nodes

![P-Rep Networking Model](../../img/p-rep1.png)

Above diagram shows how P-Rep nodes are interacting with each other in the test environment. To get access to the testnet, please read the medium post, [P-Rep TestNet Application Open](https://medium.com/helloiconworld/p-rep-testnet-application-open-97e3f7ad1e6d).




- Endpoint: https://{YOUR_GROUP_NAME}.net.solidwallet.io

  - Endpoint is the load balancer that accepts the transaction requests from DApps and relays the requests to an available P-Rep node. In the test environment, ICON foundation is running the endpoint. It is also possible for each P-Rep to setup own endpoint to directly serve DApps (as depicted in PRep-Node4), but that configuration is out of the scope of this document. 
- Tracker: https://{YOUR_GROUP_NAME}.tracker.solidwallet.io

  - A block and transaction explorer attached to the test network.
- IP List: https://download.solidwallet.io/conf/{YOUR_GROUP_NAME}_prep_iplist.json

  - ICON foundation will maintain the IP list of P-Reps. The JSON file will contain the list of IPs. You should configure your firewalls to allow in/outbound traffic from/to the IP addresses.  Following TCP ports should be open.
  - Port 7100: Used by gRPC for peer to peer communication between nodes.
  - Port 9000: Used by  JSON-RPC API server.
  - The IP whitelist will be automatically updated on a daily basis from the endpoint of the seed node inside the P-Rep Node Docker.




## Inside a P-Rep Node

**A process view of a P-Rep node**  

There are five processes, `iconrpcserver`, `iconservice`, `loopchain`, `loop-queue`, and `loop-logger`. 

![P-Rep Architecture Diagram](../../img/p-rep2.png)

- iconrpcserver

  - `iconrpcserver` handles JSON-RPC message requests
  - ICON RPC Server receives request messages from external clients and sends responses. When receiving a message, ICON RPC Server identifies the method that the request wants to invoke and transfers it to an appropriate component, either loopchain or ICON Service. 

- iconservice

  - ICON Service manages the state of ICON network (i.e., states of user accounts and SCOREs) using LevelDB.
- Before processing transactions, ICON Service performs the syntax check on the request messages and prevalidates the status of accounts to see if the transactions can be executable.
  
- loopchain

  - `loopchain` is the high-performance Blockchain Consensus & Network engine of ICON.

- loop-queue (RabbitMQ)

  - RabbitMQ is the most widely deployed open source message broker. 
  - loopchain uses RabbitMQ as a message queue for inter-process communication. 

- loop-logger (Fluentd)

  - Fluentd is the open source data collector, which lets you unify the data collection and consumption.
  - Fluentd is included in the P-Rep node image. You can use Fluentd to systemically collect and aggregate the log data that other processes produce. 



**Which ports a P-Rep Node is using?**

For external communication:

- TCP 7100: gRPC port used for peer-to-peer connection between peer nodes.
- TCP 9000: JSON-RPC or RESTful API port serving application requests.

For internal communication:

- TCP 5672: RabbitMQ port for inter-process communication.

For RabbitMQ management console:

- TCP 15672: RabbitMQ Management will listen on port 15672. 
  - You can use RabbitMQ Management by enabling this port. It must be enabled before it is used.
  - You can access the management web console at `http://{node-hostname}:15672/`



## P-Rep Installation using Docker

Please read the SW requirements above. In this chapter, we start by going through the docker installation. 

If you already have installed docker and docker compose, you can skip the part below, and directly go to the [Running P-Rep Node on Docker Container](#running-p-rep-node-on-docker-container)

### Prerequisites - Docker & Docker Compose Installation

If you don't already have docker installed, you can download it here: <https://www.docker.com/community-edition>. Installation requires sudo privilege.

#### On Centos 7

**Step 1: Install Docker**

```shell
## Install necessary packages:
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2

## Configure the docker-ce repo:
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

## Install docker-ce:
$ sudo yum install docker-ce

## Add your user to the docker group with the following command.
$ sudo usermod -aG docker $(whoami)

## Set Docker to start automatically at boot time:
$ sudo systemctl enable docker.service

## Finally, start the Docker service:
$ sudo systemctl start docker.service

## Then we'll verify docker is installed successfully by checking the version:
$ docker version 

```



**Step 2: Install Docker Compose**

```shell
## Install Extra Packages for Linux
$ sudo yum install epel-release

## Install python-pip
$ sudo yum install -y python-pip

## Then install Docker Compose:
$ sudo pip install docker-compose

## You will also need to upgrade your Python packages on CentOS 7 to get docker-compose to run successfully:
$ sudo yum upgrade python*

## To verify the successful Docker Compose installation, run:
$ docker-compose version

```



#### On Ubuntu 16.04+

**Step 1: Install Docker**

```shell
## Update the apt package index:
$ sudo apt-get update

## Install necessary packages:
$ sudo apt-get install  -y systemd apt-transport-https ca-certificates curl gnupg-agent software-properties-common 

## Add Docker's official GPG key:
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

## Add the apt repository
$ add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

## Update the apt package index:
$ sudo apt-get update

## Install docker-ce:
$ sudo apt-get -y install docker-ce docker-ce-cli containerd.io

## Add your user to the docker group with the following command.
$ sudo usermod -aG docker $(whoami)

## Set Docker to start automatically at boot time:
$ sudo systemctl enable docker.service

## Finally, start the Docker service:
$ sudo systemctl start docker.service

## Then we'll verify docker is installed successfully by checking the version:
$ docker version

```



**Step 2: Install Docker Compose**

```shell
## Install python-pip
$ sudo apt-get install -y python-pip

## Then install Docker Compose:
$ sudo pip install docker-compose

## To verify the successful Docker Compose installation, run:
$ docker-compose version

```



### Running a P-Rep Node on Docker Container

You have docker installed, then proceed the following steps to install the P-Rep node.

#### Step 1. Pull the docker image

**Pull the latest stable version of an image.**

```shell
$ docker pull iconloop/prep-node:1905292100xdd3e5a
```



#### Step 2. Run the P-Rep Node as a Docker container

**Using docker command**

```shell
$ docker run -d  -p 9000:9000 -p 7100:7100 -v ${PWD}/data:/data iconloop/prep-node:1905292100xdd3e5a
```



**Using docker-compose command (Recommended)**

Open `docker-compose.yml` in a text editor and add the following content:

```yml
version: '3' 
services:    
     container:        
          image: 'iconloop/prep-node:1905292100xdd3e5a'        
          container_name: 'prep-node'        
          volumes:            
               - ./data:/data        
          ports:           
               - 9000:9000           
               - 7100:7100
```



**Run docker-compose**

```shell
$ docker-compose up -d
```



Above command options do the followings.

1. Map container ports 7100 and 9000 to the host ports.

2. Mount a volume into the docker container.

   - `-v ${PWD}/data:/data`  sets up a bind mount volume that links the `/data/` directory from inside the P-Rep Node container to the `${PWD}/data` directory on the host machine.

   - `data` folder will have the following directory structure.

```shell
.
|---- data  
|     |---- PREP-TestNet   → Default ENV directory  
|          |---- .score_data  
|          |      |-- db      → root directory that SCOREs will be installed
|          |      |-- score   → root directory that the state DB file will be created
|          |---- .storage     → root directory that the block DB will be stored
|          |---- log          → root directory that log files will be stored
```



## P-Rep Node Operation and Configuration

### Start Node

Run docker-compose up.

```shell
$ docker-compose up -d
prep_prep_1 is up-to-date
```



The ``docker ps``  command shows the list of running docker containers. 

```shell
$ docker ps
CONTAINER ID   IMAGE                                                          COMMAND                CREATED              STATUS                          PORTS                                                                 NAMES
0de99e33cdc9     iconloop/prep-node:1905292100xdd3e5a    "/src/entrypoint.sh"      2 minutes ago        Up 2 minutes(healthy)    0.0.0.0:7100->7100/tcp, 0.0.0.0:9000->9000/tcp prep_prep_1
```

The meaning of each column in the `docker ps` result output is as follows. 

| Column       | Description                                                  |
| :----------- | :----------------------------------------------------------- |
| CONTAINER ID | Container  ID                                                |
| IMAGE        | P-Rep Node's  image name                                     |
| COMMAND      | The script will be executed whenever a P-Rep Node container is run |
| STATUS       | Healthcheck status. One of "starting" , "healthy", "unhealthy" or "none" |
| PORTS        | Exposed ports on the running container                       |
| NAMES        | Container name                                               |



You can read the container booting log from the log folder.

```shell
$ tail -f data/PREP-TestNet/log/booting_20190419.log
[2019-04-19 02:19:01.454] DEFAULT_STORAGE_PATH=/data/PREP-TestNet/.storage
[2019-04-19 02:19:01.459] scoreRootPath=/data/PREP-TestNet/.score_data/score
[2019-04-19 02:19:01.464] stateDbRootPath=/data/PREP-TestNet/.score_data/db
[2019-04-19 02:19:01.468] P-REP package version info - 1905292100xdd3e5a
[2019-04-19 02:19:02.125] iconcommons 1.0.5.2 iconrpcserver 1.3.1 iconservice 1.3.0 loopchain 2.1.7
[2019-04-19 02:19:07.107] Enable rabbitmq_management
[2019-04-19 02:19:10.676] Network: PREP-TestNet
[2019-04-19 02:19:10.682] Run loop-peer and loop-channel start
[2019-04-19 02:19:10.687] Run iconservice start!
[2019-04-19 02:19:10.692] Run iconrpcserver start!
```



### Stop Node

```shell
$ docker-compose down

Stopping prep_prep_1 ... done
Removing prep_prep_1 ... done
Removing network prep_default
```



### View Node Status

```shell
$ curl localhost:9000/api/v1/status/peer

{
    "made_block_count": 0,
    "status": "Service is online: 0",
    "state": "Vote",
    "peer_type": "0",
    "audience_count": "0",
    "consensus": "siever",
    "peer_id": "hx1787c2194f56bb550a8daba9bbaea00a4956ed58",
    "block_height": 184,
    "round": 1,
    "epoch_height": 186,
    "unconfirmed_block_height": 0,
    "total_tx": 93, 
    "unconfirmed_tx": 0,
    "peer_target": "20.20.1.195:7100",
    "leader_complaint": 185,
    "peer_count": 5,
    "leader": "hx7ff69280a1483c660695039c14ba954bb101bb66",
    "epoch_leader": "hx7ff69280a1483c660695039c14ba954bb101bb66",
    "mq": {
         "peer": {
               "message_count": 0
          },
         "channel": {
               "message_count": 0
         },
         "score": {
              "message_count": 0
         }
     }
}
```



### Docker Environment Variables

If you want change the TimeZone, open `docker-compose.yml` in a text editor and add the following content:

```yml
version: '3' services:    container:        image: 'iconloop/prep-node:1905292100xdd3e5a'        container_name: 'prep-node'        volumes:            - ./data:/data        ports:           - 9000:9000           - 7100:7100       environment:          TZ: "America/Los_Angeles"
```



The P-Rep Node image supports the following environment variables:

| Variable            | Description                                                  | Default Value          | Allowed Value                |
| :------------------ | :----------------------------------------------------------- | :--------------------- | :--------------------------- |
| TZ                  | Setting the TimeZone Environment. <br>[List of TZ name](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) | Asia/Seoul             | TZ database name             |
|                     |                                                              |                        |                              |
| IPADDR              | Setting the IP address                                       | Your public IP address | IP address                   |
| DEFAULT_PATH        | Setting the Default Root PATH                                | /data/PREP-TestNet     | PATH String                  |
| USE_MQ_ADMIN        | Enable RabbitMQ management Web interface.The management UI can be accessed using a Web browser at http://{node-hostname}:15672/. For example, for a node running on a machine with the hostname of prep-node, it can be accessed at http://prep-node:15672/ | false                  | boolean (true/false)         |
| MQ_ADMIN            | RabbitMQ management username                                 | admin                  |                              |
| MQ_PASSWORD         | RabbitMQ management password                                 | iamicon                |                              |
| LOOPCHAIN_LOG_LEVEL | loopchain Log Level                                          | INFO                   | DEBUG, INFO, WARNING, ERROR  |
| ICON_LOG_LEVEL      | iconservice Log Level                                        | INFO                   | DEBUG, INFO, WARNING, ERROR  |
| LOG_OUTPUT_TYPE     | Choose a log output type                                     | file                   | file, console, file\|console |
| RPC_WORKER          | Setting the number of RPC workers                            | 3                      | Number                       |



## Troubleshooting

### Q: How to check if container is running or not

The ``docker ps``  command shows the list of running docker containers. 

```shell
$ docker ps
CONTAINER ID   IMAGE                                                          COMMAND                CREATED              STATUS                          PORTS                                                                 NAMES
0de99e33cdc9     iconloop/prep-node:1905292100xdd3e5a    "/src/entrypoint.sh"      2 minutes ago        Up 2 minutes(healthy)    0.0.0.0:7100->7100/tcp, 0.0.0.0:9000->9000/tcp prep_prep_1
```

You should look at the `STATUS` field to see if the container is running up and in `healthy` state. 

Inside the container, there is a `healthcheck` script running with the following configuration. It will return `unhealthy` when fails. 

| Healthcheck option | value |
| :----------------- | :---- |
| retries            | 4     |
| interval           | 30s   |
| timeout            | 20s   |
| start-period       | 60s   |

The container can have three states:

- starting - container just starts
- healthy - when the health check passes
- unhealthy - when the health check fails



If the container does not start properly or went down unexpectedly, please check the `booting.log`. Below is the log messages on **success**. 

```shell
$ cat data/PREP-TestNet/log/booting_${DATE}.log 

[2019-04-19 02:19:01.435] Your IP: 20.20.1.195
[2019-04-19 02:19:01.439] RPC_PORT: 9000 / RPC_WORKER: 3
[2019-04-19 02:19:01.444] DEFAULT_PATH=/data/PREP-TestNet in Docker Container
[2019-04-19 02:19:01.449] DEFAULT_LOG_PATH=/data/PREP-TestNet/log
[2019-04-19 02:19:01.454] DEFAULT_STORAGE_PATH=/data/PREP-TestNet/.storage
[2019-04-19 02:19:01.459] scoreRootPath=/data/PREP-TestNet/.score_data/score
[2019-04-19 02:19:01.464] stateDbRootPath=/data/PREP-TestNet/.score_data/db
[2019-04-19 02:19:01.468] P-REP package version info - 1905292100xdd3e5a
[2019-04-19 02:19:02.125] iconcommons 1.0.5.2 iconrpcserver 1.3.1 iconservice 1.3.0 loopchain 2.1.7
[2019-04-19 02:19:07.107] Enable rabbitmq_management
[2019-04-19 02:19:10.676] Network: PREP-TestNet
[2019-04-19 02:19:10.682] Run loop-peer and loop-channel start
[2019-04-19 02:19:10.687] Run iconservice start!
[2019-04-19 02:19:10.692] Run iconrpcserver start!

```



### Q: How to find error

**Error log messages example**

Grep the `ERROR` messages from the log files to find the possible cause of the failure.

```shell
$ cat data/PREP-TestNet/log/booting_${DATE}.log | grep ERROR

[2019-04-19 02:08:48.746] [ERROR] Download Failed - http://20.20.1.149:5000/cert/20.20.1.195_public.der status_code=000

[2019-04-19 01:58:46.439] [ERROR] Unauthorized IP address, Please contact our support team
```



**Docker container generates below log files**

- booting.log
  - The log file contains the errors that occurred while the docker container starts up.
- iconrpcserver.log
  - The log file contains information about the request/response message handling going through the iconrpcserver. 
- iconservice.log
  - The log file contains the internals of ICON Service
- loopchain.channel-txcreator-icon_dex_broadcast.icon_dex.log
  - The log file contains information about TX broadcast from a node to other nodes
- loopchain.channel-txcreator.icon_dex.log
  - The log file contains information about the process of confirming TX
- loopchain.channel-txreceiver.icon_dex.log
  - The log file contains information about receiving the broadcasted TX from a node.
- loopchain.channel.icon_dex.log
  - The log file contains information about internals of loopchain engine



### Q: How to monitor **resources**

We recommend the following tools for resource monitoring

1. Network monitoring - iftop, nethogs, vnstat
2. CPU/Memory monitoring - top, htop
3. Disk I/O monitoring - iostat, iotop
4. Docker monitoring - docker stats, ctop
