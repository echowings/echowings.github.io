---
title: "How to Build Qemu With Cloudinit Support for Vyos"
date: 2024-02-21T16:17:56+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=d09996cd
categories:
  - Blog
tags:
  - vyos
  - qemu
slug:
  - vyos
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Perepare Linux
For me I perfer to Debian linux

## Install docker
```shell
apt install -y wget curl 
cp /etc/apt/sources.list /etc/apt/sources.list-bak
wget https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bullseye -O /etc/apt/sources.list
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
```

## Install software

```shell
apt install -y wget curl git-core
```


## Git Repository

  - git pull 
```shell
git clone https://github.com/vyos/vyos-vm-images.git
cd vyos-vm-images
```

  - commit download-iso on qemu.yaml
```qemu.yml
...
- install-packages
# - download-iso
- mount-iso
...
```


## Download vyos iso file

```shell
mkdir tmp
curl -fsSL https://xxxxxxx/vyos-1.3.6-amd64.iso -o tmp/vyos-1.3.6-amd64.iso
ls /tmp/*.iso
```

```shell
wget https://raw.githubusercontent.com/vyos/vyos-vm-images/current/Dockerfile
docker run --rm -it --privileged -v $(pwd):/vm-build -v $(pwd)/images:/images -w /vm-build vyos-vm-images:latest bash

sudo ansible-playbook qemu.yml   \
     -e disk_size=2   \
     -e iso_local=tmp/vyos-1.3.6-amd64.iso    \
     -e grub_console=serial   \
     -e cloud_init=true    \
     -e cloud_init_ds=NoCloud    \
     -e guest_agent=qemu    \
     -e enable_ssh=true
```

## copy qcow2 file to `tmp`

```shell
ls /tmp/*.qcow2
cp /tmp/vyos-1.3.6-cloud-init-2G-qemu.qcow2 tmp/
```


## Reference
  - [VyOS Qemu Image Build](https://codingpackets.com/blog/vyos-qemu-image-build/)