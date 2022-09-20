---
title: "How to Install Graylog With Docker Compose"
date: 2019-12-24T11:27:21+08:00
lastmode: 2019-12-24T11:27:21+08:00
draft: false
tags: [ "graylog", "docker","docker-compose", "linux" ]
categories: ["graylog"]
reward: true
mathjax: true
---

# How to Install Graylog with Docker-compose


|Hostname | my-graylog |
|---|---|
## Docker-compose.yaml
``` yaml
version: '2'
services:
  # MongoDB: https://hub.docker.com/_/mongo/
  mongodb:
    image: mongo:3
    volumes:
      - ./data/mongo_data:/data/db
      - /etc/localtime:/etc/localtime:ro
  # Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/6.x/docker.html
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch-oss:6.8.2
    volumes:
      - ./data/es_data:/usr/share/elasticsearch/data
      - /etc/localtime:/etc/localtime:ro
    environment:
      - http.host=0.0.0.0
      - transport.host=localhost
      - network.host=0.0.0.0
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    mem_limit: 1g
  # Graylog: https://hub.docker.com/r/graylog/graylog/
  graylog:
    image: graylog/graylog:3.1
    volumes:
      - ./data/graylog_journal:/usr/share/graylog/data/journal
      - /etc/localtime:/etc/localtime:ro
    environment:
      # CHANGE ME (must be at least 16 characters)!
      - GRAYLOG_PASSWORD_SECRET=somepasswordpepper
      # Password: admin
      - GRAYLOG_ROOT_PASSWORD_SHA2=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
      - GRAYLOG_HTTP_EXTERNAL_URI=http://my-graylog:9000/
    links:
      - mongodb:mongo
      - elasticsearch
    depends_on:
      - mongodb
      - elasticsearch
    ports:
      # Graylog web interface and REST API
      - 9000:9000
      # Syslog TCP
      - 1514:1514
      # Syslog UDP
      - 1514:1514/udp
      # GELF TCP
      - 12201:12201
      # GELF UDP
      - 12201:12201/udp
# Volumes for persisting data, see https://docs.docker.com/engine/admin/volumes/volumes/

```


## Start graylog

``` bash
docker-compose up -d
```

## Fixed permission of folder

``` bash
chown -R 1100:1100 data/graylog_journal/
chown -R 1000:1000 data/es_data/

docker-compose down
docker-compose up -d

```

