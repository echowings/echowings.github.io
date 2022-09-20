---
title: "How to Deploy Vlmcsd With Docker-Compose"
date: 2021-08-22T22:46:37+08:00
lastmode: 2021-08-22T22:46:37+08:00
draft: false
tags: [ "kms", "vlmcsd","docker", "dockerfile", "docker-compose" ]
categories: ["container"]
reward: true
mathjax: true
---

# How to deploy vlmcsd with docker-compose


```dockerfile
FROM alpine:latest as builder
WORKDIR /root
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
RUN apk add --no-cache git make build-base && \
    && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && \
    git clone --branch master --single-branch https://github.com/Wind4/vlmcsd.git && \
    cd vlmcsd && \
    make

FROM alpine:latest
COPY --from=builder /root/vlmcsd/bin/vlmcsd /vlmcsd
RUN apk add --no-cache tzdata

EXPOSE 1688/tcp

CMD ["/vlmcsd", "-D", "-d", "-t", "3", "-e", "-v"]
```

