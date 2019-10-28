---
title: "How to prevent DDoS attacks using nginx"
excerpt: "General information about the ICON P-Rep election - https://icon.community/iconsensus/"
---

This document is a guideline about how to prevent DDoS attacks using nginx.

## Intended Audience
We recommend all P-Rep candidates to go through this guideline.

## Pre-requisites
We assume that you have previous knowledge and experience in:
- IT infrastructure management
- Linux or UNIX system administration
- Network administration
- Linux server and docker service troubleshooting
- Docker container

### Introduction to Nginx
Nginx is a web server that optimizes security and speed that consists of one master process and several worker processes.
Nginx functions in an event-driven way and processes events when a new event occurs asynchronously.
It functions efficiently with a small number of threads, thus uses less CPU and requires less memory.
Using nginx in reverse proxy mode prevents DDoS attacks by throttle setting and enables whitelist based networks.


![usage_nginx](../../img/prep/usage_nginx.jpg)

### Reverse proxy advantage
The reverse proxy receives data from the internal server and sends it to the client. This prevents direct access to the internal server and acts as a relay for indirect access. The reverse proxy has many security advantages.
![usage_nginx](../../img/prep/reverse-proxy.jpg)

1. Security: External users cannot recognize the existence of servers on the actual internal network. All transmissions happen through the reverse proxy server which maps the request to the internal server information and processes accordingly. This helps to protect internal server information from the external environment.
2. ACL: Defines whether to allow or deny access from external users.
3. Log/Audit: Check the information of external users who try to access the network.

## How to use 
### Prep + Nginx composition
##### Nginx reverse package for prep node
- Please refer to the link below.
 [nginx_docker](https://github.com/JINWOO-J/nginx_docker)
 
* Configure P-Rep node software based on nginx so that each node can communicate through nginx.
* Set nginx allow ip to whitelist accessible IPs from P-Rep nodes.* Monitor the P-Rep node IP on a regular basis and renew the IP list if there is a change in allow ip. 
* Install docker in bridge mode.

![prep_using_nginx.jpg](../../img/prep/prep_using_nginx.jpg)


##### IP whitelist renewal
* One can either block all IPs or allow only specific IPs (or bands)
* Granting access to whitelisted P-Rep nodes, external attacks can be prevented.

![prep-nginx-docker.jpg](../../img/prep/prep-nginx-docker.jpg)

- Add Prep IP List to the whitelist
	- An iInternal script calls the Prep IP List(Call API) on a regular basis, creates the whitelist, and reloads nginx.
- How to add to the whitelist
	- Create `allow_ip.conf` that includes the whitelist
```nginx
#./user_conf/allow_ip.conf
    allow 10.10.10.10;
    allow 20.20.20.20;
    allow 30.30.30.30;
```
  - Add environment variable to `docker-compose.yml` environment variable
```yaml
version: '3'
services:
   nginx:
     images:
     ...
     vloumes:
        - ./user_conf/allow_ip.conf:/etc/nginx/user_conf
##################################################################
## Mount allow_ip.conf to docker volume
## File name can be random since all files in /etc/nginx/user_conf areis accessible within nginx docker
```
  - There is no service suspension in case of nginx reload. 

##### Nginx Throttle
* Throttle setting
	* Use the ‘limit_req’ module to limit excessive requests from an IP or a specific URI
	* e.g. Process 100 requests per second of an IP
	* e.g. Process exactly 100 requests per second of an IP

```nginx
# throttle setting example
limit_req_zone $binary_remote_addr zone=by_ip:10m rate=200r/s;
limit_req_zone $request_uri zone=by_uri:10m rate=200r/s;

# limit_req_zone : Declare a zone to limit the request
# binary_remote_addr : Client IP based limit
# request_uri : URI based limit
# share memory assign : 10M
# rate : If there are more than 200 requests per second, further requests will be limited
```
##### Dockerizing
```nginx
#nginx docker-compose.yml
version: '3'
services:
   prep:
      image: 'iconloop/prep-node:{tag}'
      container_name: prep
      cap_add:
         - SYS_TIME
      environment:
         ...

   nginx_throttle:
      image: 'looploy/nginx:1.17.1'
      container_name: nginx_1.17
      environment:
         NGINX_LOG_OUTPUT: 'file'
         NGINX_LOG_TYPE: 'main'
         NGINX_USER: 'root'
         VIEW_CONFIG: "yes"
         USE_NGINX_THROTTLE: "yes"
         NGINX_THROTTLE_BY_URI: "yes"
         NGINX_RATE_LIMIT: "200r/s"
         NGINX_BURST: "5"
         NGINX_SET_NODELAY: "no"
         GRPC_PROXY_MODE: "yes"
         USE_VTS_STATUS: "yes"
         TZ: "GMT-9"
         SET_REAL_IP_FROM: "0.0.0.0/0"
         PREP_MODE: "yes"
         NODE_CONTAINER_NAME: "prep"
         PREP_NGINX_ALLOWIP: "yes"
         NGINX_ALLOW_IP: "0.0.0.0/0"
         NGINX_LOG_FORMAT: '$$realip_remote_addr $$remote_addr  $$remote_user [$$time_local] $$request $$status $$body_bytes_sent $$http_referer "$$http_user_agent" $$http_x_forwarded_for $$request_body'
      volumes:
         - ./data/loopchain/nginx:/var/log/nginx
         - ./user_conf:/etc/nginx/user_conf
      ports:
         - '7100:7100'
         - '9000:9000'
```
##### Environment variable


| Environment variable | Description|Default value| Allowed value|
|--------|--------|-------|-------|
 TRACKER\_IPLIST| Required for tracker to monitor prep|15.164.151.101 15.164.183.120 52.79.145.149 54.180.178.129 ||
 PREP\_NGINX\_ALLOWIP| `no` :  Set allow come to anyone. `yes`: Set nginx allow ip to whitelist accessible IPs from P|no ||
 PREP\_MODE| PREP\_MODE mode whitelist based nginx usage|no |   (yes/no)|
 NODE\_CONTAINER\_NAME| container name in order to connect to prep|prep ||
 PREP\_LISTEN\_PORT| Choose a prep|9000 ||
 PREP\_PROXY\_PASS\_ENDPOINT| prep's container name for RPC API  (if you selected `PREP\_MODE`, Required input)|http||
 PREP\_NODE\_LIST\_API| In order to get prep's white ip list, ENDPOINT API URL (Required input)|${PREP\_PROXY\_PASS\_ENDPOINT/api/v3 ||
 USE\_DOCKERIZE| `go template` usage ( yes/no )|yes  ||
 VIEW\_CONFIG| Config print at launch ( yes/no )|no       ||
 UPSTREAM| upstream setting|localhost||
 DOMAIN| domain setting|localhost          ||
 LOCATION|ADD\_LOCATION|| additional location setting|
 WEBROOT| webroot setting|/var/www/public  ||
 NGINX\_EXTRACONF| additional conf settings| ||
 USE\_DEFAULT\_SERVER| nginx's default conf setting|no  ||
 USE\_DEFAULT\_SERVER\_CONF| nginx's default server conf setting| ||
 NGINX\_USER|www|wwwdata  ||
 NGINX\_SET\_NODELAY| Delay option if rate limit is exceeded|no  | ( yes/no )|
 WEB\_SOCKET\_URIS| URI for using nginx as a websocket proxy|/api/ws/* /api/node/* ||
 NUMBER\_PROC| worker processes count|$(nproc)  |  max number of processes|
 WORKER\_CONNECTIONS| setting WORKER\_CONNECTIONS|4096  ||
 GRPC\_LISTEN\_PORT| Used by gRPC Listen port|7100 ||
 LISTEN\_PORT|${GRPC\_LISTEN\_PORT}|${GRPC\_LISTEN\_PORT||
 SENDFILE|on|on||
 SERVER\_TOKENS|off|off||
 KEEPALIVE\_TIMEOUT|65|65||
 KEEPALIVE\_REQUESTS|15|15||
 TCP\_NODELAY|on|on||
 TCP\_NOPUSH|on|on||
 CLIENT\_BODY\_BUFFER\_SIZE|3m|3m||
 CLIENT\_HEADER\_BUFFER\_SIZE|16k|16k||
 CLIENT\_MAX\_BODY\_SIZE|100m|100m||
 FASTCGI\_BUFFER\_SIZE|256K|256K||
 FASTCGI\_BUFFERS|8192 4k|8192 4k||
 FASTCGI\_READ\_TIMEOUT|60|60||
 FASTCGI\_SEND\_TIMEOUT|60|60||
 TYPES\_HASH\_MAX\_SIZE|2048|2048||
 NGINX\_LOG\_TYPE| output log format type|default  |  (json/default)|
 NGINX\_LOG\_FORMAT|  '$realip\_remote\_addr $remote\_addr|   ||
 NGINX\_LOG\_OUTPUT| output log type|file | stdout or file  or off|
 USE\_VTS\_STATUS| vts monitoring usage|yes   | (yes/no)|
 USE\_NGINX\_STATUS| nginx status monitoring usage|yes |(yes/no)|
 NGINX\_STATUS\_URI| nginx\_status URI|nginx\_status ||
 NGINX\_STATUS\_URI\_ALLOWIP| nginx\_status URI is only allow requests from this IP address|127.0.0.1 ||
 USE\_PHP\_STATUS|no|no||
 PHP\_STATUS\_URI|php\_status|php\_status||
 PHP\_STATUS\_URI\_ALLOWIP|127.0.0.1|127.0.0.1||
 PRIORTY\_RULE|allow|allow||
 NGINX\_ALLOW\_IP| Administrator IP addr for detail monitoring|    ||
 NGINX\_DENY\_IP||||
 NGINX\_LOG\_OFF\_URI||||
 NGINX\_LOG\_OFF\_STATUS||||
 DEFAULT\_EXT\_LOCATION| extension setting  ~/.jsp ~/.php|php  ||
 PROXY\_MODE| gRPC proxy mode usage|no   | (yes/no)|
 GRPC\_PROXY\_MODE| gRPC proxy mode usage|no | (yes/no)|
 USE\_NGINX\_THROTTLE| rate limit usage|no |  (yes/no)|
 NGINX\_THROTTLE\_BY\_URI| URI based rate limit usage (yes/no)|no ||
 NGINX\_THROTTLE\_BY\_IP| IP based rate limit usage (yes/no)|no  ||
 PROXY\_PASS\_ENDPOINT| proxy endporint of gRPC|grpc||
 NGINX\_ZONE\_MEMORY| Sets the shared memory zone for `rate limit`|10m    ||
 NGINX\_RATE\_LIMIT| rate limiting value|100r/s   ||
 NGINX\_BURST|Excessive requests are delayed until their number exceeds the maximum burst size,  maximum queue value ( If the value is `10`, apply from `11`)|10                 ||
 SET\_REAL\_IP\_FROM| SET\_REAL\_IP\_FROM|0.0.0.0/0   ||
 NGINX\_PROXY\_TIMEOUT|90|90||

