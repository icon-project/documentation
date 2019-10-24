## 1. Export your keystore file first
   Export your Keystore file whitch you registered in. You can see how to get the Keystore file from this [link.](https://www.icondev.io/docs/account-management)


## 2. Place your keystore file in the proper path
The keystore file needs to be exported and stored in the cert directory. Below is a directory structure under the `docker-compose.yml`  

```
|-- docker-compose.yml   
|-- data  → data directory            
    |-- mainnet  → block DB directory 
    |-- log  → log directory
|-- cert  → keytore or cert key directory
    |-- YOUR_KEYSTORE_FILE  → put your keystore file
```

Important - you should put your keystore file under the `/cert` folder.


## 3. Rename Keystore file simply
Enter a short name for the keystore file (recommended).  

## 4. Import your keystore file into `docker-compose.yml`

If your keystore file name is `textKeystore` then the `docker-compose.yml` file looks like this:


`
$ cat docker-compose.yml
`


```yml

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
        PRIVATE_KEY_FILENAME: "testKeystore" # only file name
        PRIVATE_PASSWORD: "testKeystorepassword"
    cap_add:
        - SYS_TIME      
    volumes:
        - ./data:/data # mount a data volumes
        - ./cert:/cert # Automatically generate cert key files here
    ports:
        - 9000:9000
        - 7100:7100
```

Also, you can see the directory path as below:

```
|-- docker-compose.yml   
|-- data  → data directory            
    |-- mainnet  → block DB directory 
    |-- log  → log directory
|-- cert  → keytore or cert key directory
    |-- testKeystore  → put your keystore file
```



## Troubleshooting

### Q: How to check if container is running or not

The ``docker ps``  command shows the list of running docker containers. 

```shell
$ docker ps
CONTAINER ID   IMAGE                                                          COMMAND                CREATED              STATUS                          PORTS                                                                 NAMES
0de99e33cdc9     iconloop/prep-node:1910211829xc2286d    "/src/entrypoint.sh"      2 minutes ago        Up 2 minutes(healthy)    0.0.0.0:7100->7100/tcp, 0.0.0.0:9000->9000/tcp prep_prep_1
```

You should look at the `STATUS` field to see if the container is running up and in `healthy` state. 

Inside the container, there is a `healthcheck` script running with the following configuration. It will return `unhealthy` when it fails. 

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
$ cat data/PREP-MainNet/log/booting_${DATE}.log 

[2019-10-23 17:47:05.204] Your IP: xxx.xxx.xxx.xxx
[2019-10-23 17:47:05.209] RPC_PORT: 9000 / RPC_WORKER: 3
[2019-10-23 17:47:05.214] DEFAULT_PATH=/data/mainnet in Docker Container
[2019-10-23 17:47:05.219] DEFAULT_LOG_PATH=/data/mainnet/log
[2019-10-23 17:47:05.224] DEFAULT_STORAGE_PATH=/data/mainnet/.storage
[2019-10-23 17:47:05.229] scoreRootPath=/data/mainnet/.score_data/score
[2019-10-23 17:47:05.234] stateDbRootPath=/data/mainnet/.score_data/db
[2019-10-23 17:47:05.239] Time synchronization with NTP / NTP SERVER: time.google.com
[2019-10-23 17:47:12.022] P-REP package version info - _1910211829xc2286d
[2019-10-23 17:47:12.697] iconcommons             1.1.2
iconrpcserver           1.4.5
iconsdk                 1.2.0
iconservice             1.5.15
loopchain               2.4.15

```



### Q: How to find error

**Error log messages example**

Grep the `ERROR` messages from the log files to find the possible cause of the failure.

```shell
$ cat data/PREP-MainNet/log/booting_${DATE}.log | grep ERROR

[2019-08-12 02:08:48.746] [ERROR] Download Failed - http://20.20.1.149:5000/cert/20.20.1.195_public.der status_code=000

[2019-08-12 01:58:46.439] [ERROR] Unauthorized IP address, Please contact our support team
```



**Docker container generates below log files**

- booting.log
  - The log file contains the errors that occurred when the docker container starts up.
- iconrpcserver.log
  - The log file contains information about the request/response message handling going through the iconrpcserver. 
- iconservice.log
  - The log file contains the internals of ICON Service
- loopchain.channel-txcreator-icon_dex_broadcast.icon_dex.log
  - The log file contains information about TX broadcast from a node to other nodes
- loopchain.channel-txcreator.icon_dex.log
  - The log file contains information about the process of confirming TXƒ
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