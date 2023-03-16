---
title: "how to install docker and docker-compose on debian 11 or proxmox ve 7"
date: 2023-03-16T17:32:08+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=0bbdeb62
categories:
  - containers
tags:
  - docker
  - docker-compose
  - debian
  - debian 11
  - pve
slug:
  - docker
layout: 
  - post

license: CC BY-NC-ND
---

# how to install docker and docker-compose on debian 11 or proxmox ve 7

```bash

apt-get update
apt-get install -y sudo 
# append the required Docker dependencies to the Debian system using the following command:
sudo apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common

#Add Docker’s official GPG key:
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

#Step 3: Add stable repository
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

#Step 4: Update Package Cache
apt-get update

#Step 5: Install Docker Engine
apt-get install  -y docker-ce docker-ce-cli containerd.io
#Step 6: Check Docker Version
docker --Version

#Step 7: Check Docker Service
sudo systemctl status docker


## Install Docker Compose


sudo apt update
sudo apt install -y curl wget
curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url  | grep docker-compose-linux-x86_64 | cut -d '"' -f 4 | wget -qi -
chmod +x docker-compose-linux-x86_64
mv docker-compose-linux-x86_64 /usr/bin/docker-compose

# Check Version
docker-compose version
```

## Reference

- [How to install Docker on Debian 11](https://www.fosslinux.com/49959/install-docker-on-debian.htm)
- [How To Install Latest Docker Compose on Linux
  ](https://computingforgeeks.com/how-to-install-latest-docker-compose-on-linux/)



