---
title: "How to Install Gitea With Docker-Compose"
date: 2021-03-31T10:49:33+08:00
lastmode: 2021-03-31T10:49:33+08:00
draft: false
tags: [ "docker", "docker-compose","git", "gitea", "debian", "debian 10" ]
categories: ["devops"]
reward: true
mathjax: true
---

# How to Install Gitea with Docker-Compose



```bash
cat << EOF >> docker-compose.yaml
version: "3"

networks:
  gitea:
    external: false

services:
  server:
    image: gitea/gitea:1.13.6
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=mysql
      - DB_HOST=db:3306
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
       - "3000:3000"
       - "222:22"
    depends_on:
       - db

  db:
     image: mysql:5.7
     restart: always
     environment:
       - MYSQL_ROOT_PASSWORD=gitea
       - MYSQL_USER=gitea
       - MYSQL_PASSWORD=gitea
       - MYSQL_DATABASE=gitea
     networks:
       - gitea
     volumes:
       - ./mysql:/var/lib/mysql
EOF
```

## Run it

```bash
docker-compose pull
docker-compose up -d
```

## Setup 

 - [http://xa-gitea:3000](http://xa-gitea:3000)