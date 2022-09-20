---
title: "Proxmov Ve How to Upgrade Pve"
date: 2019-04-16T12:12:58+08:00
tags: ["PVE"]
categories: [ "virtualization" ]
author: "steve dong"
draft: false
reward: true

---

# How to ugrade Proxmove VE

If your proxmox ve server in china ,Change utsc mirrors will speed up your proxmoxve upgrade speed.

Here are the steps.

## 1. Change debian,pve,ceph upgrade source, and upgrade your PVE


``` shell
#chang debian source list
echo "deb https://mirrors.ustc.edu.cn/debian/ stretch main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ stretch-updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch-updates main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ stretch-backports main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch-backports main contrib non-free

deb https://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free" > /etc/apt/sources.list

#change pve 5.x update source list to no subscription
echo "deb http://mirrors.ustc.edu.cn/proxmox/debian/pve/ stretch pve-no-subscription" > /etc/apt/sources.list.d/pve-no-sub.list
 
# Disable pve 5.x enterprise subscription
sed -i.bak 's|deb https://enterprise.proxmox.com/debian stretch pve-enterprise|\# deb https://enterprise.proxmox.com/debian stretch pve-enterprise|' /etc/apt/sources.list.d/pve-enterprise.list

#change ceph source list
echo "deb http://mirrors.ustc.edu.cn/proxmox/debian/ceph-luminous stretch main" >> /etc/apt/sources.list.d/ceph.list

# add key
wget http://mirrors.ustc.edu.cn/proxmox/debian/proxmox-ve-release-5.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-5.x.gpg

apt updat && apt -y dis-ugprade
update-grub

```
## 2. Change settings after upgrade

This step to change pve upgrade to no subscription and ceph soucre .


```shell

#change pve 5.x update source list
echo "deb http://mirrors.ustc.edu.cn/proxmox/debian/pve/ stretch pve-no-subscription" > /etc/apt/sources.list.d/pve-enterprise.list

#change ceph source list
echo "deb http://mirrors.ustc.edu.cn/proxmox/debian/ceph-luminous stretch main" > /etc/apt/sources.list.d/ceph.list

# add key
wget http://mirrors.ustc.edu.cn/proxmox/debian/proxmox-ve-release-5.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-5.x.gpg




```


