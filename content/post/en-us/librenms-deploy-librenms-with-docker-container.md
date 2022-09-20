---
title: "Librenms Deploy Librenms With Docker Container"
date: 2019-04-16T16:56:08+08:00
draft: false
tags: ["docker","librenms", "docker-compose","monitoring"]
categories: ["docker"]
auther: "steve dong"
reward: false
---

# How to deploy librenms monitoring system with docker container
Librenms is a very light and easy handle moniting software. It is suitable for us to monitoring server,networdevice, printer etc via snmp.

At now, docker is easy for software deployment. After deploy librenms in ubuntu vm, I have a try to deploy librenms with docker-compose.

## Install librenms with docker-compose

```shell
mkdir -p librenms-docker-compose
cd ./librenms-docker-compose
wget https://raw.githubusercontent.com/librenms/docker/master/examples/compose/.env
wget https://raw.githubusercontent.com/librenms/docker/master/examples/compose/docker-compose.yml
wget https://raw.githubusercontent.com/librenms/docker/master/examples/compose/librenms.env
touch acme.json
chmod 600 acme.json


```

### Change `docker-compose.yml`

`vi docker-compose.yml`


  - change `services | traefik | command ` section.

  ```shell
  	   - "--docker.domain=mydomain.net"
```


  - Change `services | `
 
|KEY| VALUE |
| --- |  --- |
| hostname   | mylibrenms |
| domainname   | mydomain.net |


  - Chang `syslog-ng` section


| KEY | VALUE |
|  --- | --- |
|hostname|mylibrenms|
|domainname|mydomain.net|
   
   






```shell
docker-compose up -d
docker-compose logs -f
```




