---
title: "How to Change Settings After Clone Debian 10 With Cloudinit Enable on Proxmox Ve"
date: 2019-11-27T11:56:41+08:00
lastmode: 2019-11-27T11:56:41+08:00
draft: false
tags: [ "pve", "debian", "docker" ]
categories: ["linux"]
reward: true
mathjax: true
---

# How to Change Settings After Clone Debian 10 With Cloudinit Enable on Proxmox VE


## Issue

I found an issue about create a linux vm as a template with cloudinit enabled. Seem like after cloned the virtual machine, the cloud-init cd rom didn't be created automatically. We need to delete it,then change setings again to apply the apt source.

## solution


```bash

#!/usr/bin/env sh

mv /etc/apt/sources.list /etc/apt/sources.list.backup
wget https://mirrors.ustc.edu.cn/repogen/conf/debian-http-4-buster -O /etc/apt/sources.list
apt update && apt -y dist-upgrade

apt install -y openssh-server git vim cloud-init apt-transport-https
sed -i 's/http/https/g' /etc/apt/sources.list
apt update && apt -y dist-upgrade

mv /etc/apt/sources.list /etc/apt/sources.list.backup
wget https://mirrors.ustc.edu.cn/repogen/conf/debian-http-4-buster -O /etc/apt/sources.list
apt update && apt -y dist-upgrade

apt install -y openssh-server git vim cloud-init apt-transport-https
sed -i 's/http/https/g' /etc/apt/sources.list
apt update && apt -y dist-upgrade

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

sed -i 's/download.docker.com/mirrors.ustc.edu.cn\/docker-ce/g' /etc/apt/sources.list

cat << EOF > /etc/default/docker
DOCKER_OPTS="--registry-mirror=https://docker.mirrors.ustc.edu.cn/"
EOF
systemctl restart docker

sudo sed -i 's/PasswordAuthentication\ no/PasswordAuthentication\ yes/g' /etc/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd

sudo sed -i 's/#PermitRootLogin\ prohibit-password/PermitRootLogin\ yes/g' /etc/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd

```