---
title: "How to Install Jenkins With Docker Compose"
date: 2019-12-10T22:39:01+08:00
lastmode: 2019-12-10T22:39:01+08:00
draft: false
tags: [ "devops", "jenkins","pipline", "docker"]
categories: ["devops"]
reward: true
mathjax: true
---

# How to install Jenkins with docker compose


## Create Dockerfile

With office jenkins docker images.change the jenkins mirrors and change alpine linux soure list to aliyun

```yaml
#Dockerfile
FROM jenkins/jenkins:alpine
USER root
ENV JENKINS_MIRROR https://mirrors.tuna.tsinghua.edu.cn
ENV JENKINS_UC_DOWNLOAD="https://mirrors.tuna.tsinghua.edu.cn/jenkins/"
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
	 && apk update \
    && apk add docker nodejs nodejs-npm yarn


```
## Create docker-compose.yaml
``` yaml
# docker-compose.yaml
version: "2.4"
services:
  jenkins:
    build: ./
    #image: jenkins/jenkins:alpine
    container_name: jenkins
    ports:
      - "80:8080"
      - "50000:50000"
    environment:
      #- JAVA_OPTS=-Xmx1500m -Duser.timezone=GMT+8
      - JAVA_OPTS=-Duser.timezone=GMT+8
      - JENKINS_UC_DOWNLOAD=https://mirrors.tuna.tsinghua.edu.cn/jenkins/
    volumes:
      - ./data/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /root/.ssh/:/root/.ssh
      - /etc/localtime:/etc/localtime:ro
#    cpus: 3.5
#    mem_limit: 1800m
    user: root
    restart: always
```

## Install Jenkins with

Check initial Admin Password by run command as show below

```bash
cat data/jenkins/secrets/initialAdminPassword
```


## Reference

  - [Jenkins for Docker 跳过插件安装及插件加速镜像设置](https://6xyun.cn/article/92)
