---
title: "How to Install Pbs2.3 on Debian11"
date: 2023-03-24T11:10:21+08:00
draft: true
image: https://picsum.photos/800/600.webp?random=244bc190
categories:
  - Blog
tags:
  - proxmox ve
  - pve
  - proxmox
  - linux
  - backup
  - debian
  - pbs
  - proxmox backup server
slug:
  - pbs
layout: 
  - post


license: CC BY-NC-ND
---

# How-to-install-PBS-on-debian-11

## Change network name from enp* to eth*

```bash
cp /etc/default/grub /etc/default/grub-bak
sed -i '/GRUB_CMDLINE_LINUX=/s/"$/ net.ifnames=0 biosdevname=0"/' /etc/default/grub
sed -i 's/enp1s0/eth0/' /etc/network/interfaces
```
Then reboot your server


## change sourcelist to utsc


```bash
cp /etc/apt/sources.list /etc/apt/sources.list-bak
wget https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bullseye -O /etc/apt/sources.list
cat /etc/apt/sources.list
apt update && apt -y dist-upgrade

apt install -y neovim net-tools
```



```bash


ipv4_addr=$(ip -4 route get 8.8.8.8 | awk {'print $7'} | tr -d '\n')
echo $ipv4_addr
sed -i "s/127.0.1.1/${ipv4_addr}/g" /etc/hosts
cat /etc/hosts 

hostname --ip-address




## Add psb no subscriont list

## Option 1: Offical mirrors
tee   /etc/apt/sources.list.d/pbs-no-subscription.list << "EOF"
deb http://download.proxmox.com/debian/pbs bullseye pbs-no-subscription
EOF


## Option 2: utsc mirros
tee   /etc/apt/sources.list.d/pbs-no-subscription.list << "EOF"
deb https://mirrors.ustc.edu.cn/proxmox/debian/pbs bullseye pbs-no-subscription
EOF


apt update 
apt -y dist-upgrade

## Install PBS server
apt install proxmox-backup

# Remove enterprise sourcelist
rm -rf /etc/apt/sources.list.d/pbs-enterprise.list

## Disable subscription alert
sed -i.backup -z "s/res === null || res === undefined || \!res || res\n\t\t\t.data.status.toLowerCase() \!== 'active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart proxmox-backup-proxy
```



## Reference
  1. [How to Install Proxmox Backup Server on Debian 11](https://www.vultr.com/docs/how-to-install-proxmox-backup-server-on-debian-11-2488/)
  - [How to: Remove “You do not have a valid subscription for this server….” from Proxmox Backup Server (PBS) 1.0-1 to 2.3-2 and up](https://dannyda.com/2020/11/13/how-to-remove-you-do-not-have-a-valid-subscription-for-this-server-from-proxmox-backup-server-pbs-1-0-1/)

