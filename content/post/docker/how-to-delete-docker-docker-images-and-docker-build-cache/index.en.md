---
title: "How to Delete Docker Docker Images and Docker Build Cache"
date: 2023-12-22T22:40:03+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=450f0e4f
categories:
  - Blog
tags:
  - docker
slug:
  - docker
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

# How to delete docker, docker image and docker build

```shell
#Delete all docker container running instance
docker rm -rf $(docker ps -aq)
 
# Delete docker images
docker rmi -f $(docker images -aq)

# Delte docker build cache
docker builder prune  -af
```