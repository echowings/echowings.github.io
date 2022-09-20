---
title: "How to Install driver of Mellanox ConnetcX 3 10Gbps Network Adpater in Promxox-VE 5.x or Debian 9"
date: 2019-06-05T11:52:23+08:00
draft: false
tags: ["pve", "storage","networking", "mellanox"]
categories: ["virtualization","networking"]
reward: true
---

# How to install dirver of mellanox ConnectX 3 10Gbps netowrk adapter in proxmox-ve 5.x or diban 9

To 



## Install dependencies software

```shell
#dependencies
apt-get -y install debhelper autotools-dev dkms zlib1g-dev python
wget quilt python-libxml2 swig dpatch graphviz chrpath pve-headers
```


## Download driver for mellanox connectx 3 for debian 9.6

``` shell
wget http://content.mellanox.com/ofed/MLNX_EN-4.6-1.0.1.1/mlnx-en-4.6-1.0.1.1-debian9.6-x86_64.tgz
```

## Install drivers

```shell
tar -zxvf mlnx-en-4.6-1.0.1.1-debian9.6-x86_64.tgz
cd mlnx-en-4.6-1.0.1.1-debian9.6-x86_64
chmod +x install
./install --skip-distro-check


```


## Swell


  - [Mellanox EN Driver for Linux
](http://www.mellanox.com/page/products_dyn?product_family=27)
  - [How-to Guide Install Mellanox Driver Proxmox VE 5.1](https://forums.servethehome.com/index.php?resources/install-mellanox-driver-proxmox-ve-5-1.35/)