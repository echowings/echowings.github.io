---
title: "How to Build a Kms Server"
date: 2019-04-11T00:17:37+08:00
draft: false
tags: ["docker","kms"]
categories: ["docker"]
auther: "steve dong"
reward: false
---

# How to build a kms server
## Option 1: docker mode

  ```shell
  docker pull luodaoyi/kms-server
  
  docker run -itd -p 1688:1688 --name kms luodaoyi/kms-server
  
  docker update --restart=always $dockername
  
  ```
 
## Option 2: docker compose 
 
 docker-compose.yml
 
```yml
 version: '3'
services:
    frontend:
        image: luodaoyi/kms-server
        container_name: mykms
        ports:
            - 1688:1688
        restart: always

 
```
 
 docker-compose start the container
 
```shell
 docker-compose up -d
```


## Active Script


``` cmd
echo "To active office"
cd "%programfiles%\Microsoft Office\Office16"
cscript ospp.vbs /sethst:xx.xx.xx.xx
cscript ospp.vbs /act

echo "to active windows"
slmgr /skms xxx.xxx.xxx.xxx
slmgr /ato

```


## Reference

  - [blog](https://blog.csdn.net/flyhorstar/article/details/87269141)
