---
title: "How to Install Elasticflow"
date: 2019-11-19T17:11:27+08:00
lastmode: 2019-11-19T17:11:27+08:00
draft: false
tags: [ "elasticflow", "docker","docker-compose", "netflow", "networking" ]
categories: ["networking"]
reward: true
mathjax: true
---


# How to install Elasticflow

Today, i found a good software that is [Elasticflow](https://github.com/robcowart/elastiflow). It is so perfect for us to monitor status of our networking. So I tried to install it will docker-compose way. You know what, after servial hours, fixed a lot bugs it works.

![](https://user-images.githubusercontent.com/10326954/57181284-fc141a80-6e91-11e9-9ec5-d0864c25a088.png)

## Download and modified `docker-compose.yml` for elasticflow

```bash
# Download `docker-compose.yml` for elasticflow
wget https://raw.githubusercontent.com/robcowart/elastiflow/master/docker-compose.yml

# version 3.5.2 is now existed. So we need change it to `3.5.1`
sed -i 's/robcowart/elastiflow-logstash-oss:3.5.2/robcowart/elastiflow-logstash-oss:3.5.1/g' -i docker-comopse.yml

#Change volumes from `/var/lib/elastiflow_es to current folder`
sed -i 's/\/var\/lib\/elastiflow_es/.\/elastiflow_es/g'  docker-compose.yml

# Create data folder and change folder permission.
mkdir elastiflow_es
chown -R 1000:1000 ./elastiflow_es

``` 

## Up and run

```bash
docker-compose up -d 
```

Then we can access elasticflow via browser
[http://\<elastciflow_host_ip\>:5601](http://<elastciflow_host_ip>:5601)

## Swell


 - [ElastiFlow™](https://github.com/robcowart/elastiflow)
 - [Problem with elasticsearch-oss #428](https://github.com/robcowart/elastiflow/issues/428)
 - [Running ElastiFlow™ on Docker](https://github.com/robcowart/elastiflow/blob/master/DOCKER.md#prepare-the-data-path)
