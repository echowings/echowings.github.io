---
title: "How to Deploy Librenms With Docker-compose"
date: 2019-06-05T21:47:54+08:00
draft: false
tags: ["librenms"]
categories: ["monitoring"]
reward: true
---

# How to deploy librenms with docker-compsoe

Librenms is a very light and easy handle moniting software. It is suitable for us to monitoring server,network device, printers etc. via snmp.

At now, docker is easy for software deployment. After deploy librenms in ubuntu vm, I have a try to deploy librenms with docker-compose.

## Download config files


```shell
mkdir librenms-docker-compose
cd librenms-docker-compose
#Download .env file
wget https://raw.githubusercontent.com/librenms/docker/master/examples/compose/.env

# Download docker-compose file
wget https://raw.githubusercontent.com/librenms/docker/master/examples/compose/docker-compose.yml

# Download librenms.env file
wget https://raw.githubusercontent.com/librenms/docker/master/examples/compose/librenms.env

touch acme.json
chmod 600 acme.json


```
## librenms server configuration
As show below , I will deploy a librenms via docker  in my lan network. the host name is `mylibrenms`,my local domain name is `private.local`.

|Type   | value |
|---|---|
|Librenms host name | `mylibrenms`|
|Domain name | `private.local`|
| Name | `myname`|
|mail account | `myname@private.local`|
|snmp community | `public` |



## Modify `.env`

```shell
vi .env
```
Change configure :

```shell
MYSQL_DATABASE=mylibrenms
MYSQL_USER=mylibrenms
MYSQL_PASSWORD=mylibrenmspassword

SMTP_SERVER=smtp.private.local
SMTP_USERNAME=myname@private.local
SMTP_PASSWORD=myPassword

TZ=Asia/Chongqing
PUID=1000
PGID=1000
```




## Modify `docker-compose.yml`

```shell
vi docker-compose.yml
```
### Change treafik config
  - Change `services: | treafik: | command: | - "--docker.domain="` to `--docker.domain=private.local`

  ```bash
sed -i 's/\"--docker.domain=example.com\"/\"--docker.domain=private.local\"/g' docker-compose.yml
```
  - Change `services: | treafik: | command: | - "--acme.email="` to `- "--acme.email="myname@private.local"`
  
  ```bash
  sed -i 's/\"--acme.email=webmaster@example.com\"/\"--acme.email=myname@private.local\"/g' docker-compose.yml
  ```
  
  
### Change smtp config

  - Change `services: | smtp: | environment: | - "SERVER_HOSTNAME="` to `- "SERVER_HOSTNAME=mylibrenms.private.local"`

  ```bash
	sed -i 's/-\ \"SERVER_HOSTNAME=librenms.example.com\"/-\ \"SERVER_HOSTNAME=mylibrenms.private.local\"/g' docker-compose.yml
```
  

### Change librenms config
  - Change `services: | librenms: | domainname: ` to `domainname: private.local`.
 
  ```bash
sed -i 's/domainname:\ example.com/domainname:\ private.local/g' docker-compose.yml

  ```
  
  - Change `services: | librenms: | hostname: ` to `hostname; mylibrenms`.
  

  ```bash
  sed -i 's/hostname:\ librenms/hostname:\ mylibrenms/g' docker-compose.yml
  
  ```
  
  - Change `services: | librenms: | labels: | - " traefik.frontend.rule="` to ` - "traefik.frontend.rule=Host:mylibrenms"`.
  
  ```bash
sed -i 's/-\ \"traefik.frontend.rule=Host:librenms.example.com\"/-\ \"traefik.frontend.rule=Host:mylibrenms.private.local\"/g' docker-compose.yml
  ```
  

### Change cron config
  - Change `services: | cron: | domainname:` to `domainname: private.local`.



  
  - Change `services: | cron: | hostname:` to `hostname: mylibrenms`.

### Change syslog-ng:

  1. Change `services: | syslog-ng: | domainname: ` to `domainname: private.local`.
  2. Change `services: | syslog-ng: | hostname:` to `hostname: mylibrenms`.

## Modify `librenms.env`


  - Change `LIBRENMS_SNMP_COMMUNITY=` to `LIBRENMS_SNMP_COMMUNITY=public`.
 
 ```bash
 sed -i 's/LIBRENMS_SNMP_COMMUNITY=librenmsdocker/LIBRENMS_SNMP_COMMUNITY=public/g' librenms.env
 ```
 
 

  
## Run docker-compose to up librenms containers

```shell
docker-compose up -d

```


### enable proxmox app moniting

``` shell

cat << EOF > librenms/config.php
<?php
$config['enable_proxmox'] = true;
EOF

```
