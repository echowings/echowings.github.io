---
title: "How to Deploy Docker in Debian 10"
date: 2019-07-19T10:52:56+08:00
tags: [ "debian", "docker", "docker-compose", "linux" ]
categories: ["linux"]
draft: false
---

# How to deploy docker in debian 10




```bash
cat << EOF > /etc/docker/aemon.json\n{\n  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]\n}\n\nEOF
systemctl retart docker
systemctl restart docker
```



## Install docker-compose

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
