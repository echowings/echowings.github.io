---
title: "How to install ProxmoxVE 5.x on Debian 9"
date: 2018-12-01T00:02:28+08:00
lastmod: 2018-12-26T01:52:23+08:00
draft: false
tags: ["PVE", "debian"]
categories: ["virtualization"]
author: "Steve Dong"
reward: true
---

# How to Install Proxmox VE 5.x on Debian 9



## Chang IP ADDRESS 

`vi /etc/network/interfaces`

``` bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).
source /etc/network/interfaces.d/*
# The loopback network interface
auto lo
iface lo inet loopback
# The primary network interface
#allow-hotplug eno1
auto eno1
#iface eno1 inet dhcp
iface eno1 inet static
address *.*.*.*
netmask *.*.*.*
gateway *.*.*.* 

```

## Change DNS

`vi /etc/resolv.conf`

## Change your ip and hostname


`vi /etc/hosts`

``` bash
127.0.0.1       localhost.localdomain localhost
10.32.0.60   xa-sys-pve02 pvelocalhost

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters



```

## Perl ALL_LC ERROR

``` bash
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
#locale-gen en_US.UTF-8
cat >> /etc/environment <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
EOF


```

## Add Utsc mirrors source

before to chang source need to install 

``` bash
apt install apt-transport-https
```


``` bash
#chang debian source list
echo "deb https://mirrors.ustc.edu.cn/debian/ stretch main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ stretch-updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch-updates main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ stretch-backports main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch-backports main contrib non-free

deb https://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free" > /etc/apt/sources.list

#change pve 5.x update source list
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve/ stretch pve-no-subscription" >> /etc/apt/sources.list

#change ceph source list to utsc mirrors
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-luminous stretch main" >> /etc/apt/sources.list

# add key
wget http://mirrors.ustc.edu.cn/proxmox/debian/proxmox-ve-release-5.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-5.x.gpg

```

## Install proxmox VE

``` bash
apt update && apt -y dist-ugprade
apt install proxmox-ve postfix open-iscsi
update-grub

```
## Remove “No valid Subscription” Message

* shell solution.

``` bash
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
	
```
* manual solution:

```bash
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
nano proxmoxlib.js

# change 
if (data.status !== 'Active') {

if (false) {

#restart pveproxy.service
systemctl restart pveproxy.service
```

# Remove sourcelist of pve enterprise subscription and ceph

``` shell
# Delete Enterprise subscription.
rm -rf /etc/apt/sources.list.d/pve-enterprise.list

# Delete proxmox ve offical ceph source. 
rm -rf /etc/apt/sources.list.d/ceph.list


```

## Remove “No valid Subscription” Message

* shell solution.

``` bash
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
	
```
* manual solution:

```bash
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
nano proxmoxlib.js

# change 
if (data.status !== 'Active') {

if (false) {

#restart pveproxy.service
systemctl restart pveproxy.service
```



## Q&A

* When you got error like this, please make sure hosts file is set without wrong. like this`10.32.0.60   xa-sys-pve02 xa-sys-pve02 pvelocalhost` 

``` bash

Setting up pve-manager (5.2-1) ...
Job for pvestatd.service failed because the control process exited with error code.
See "systemctl status pvestatd.service" and "journalctl -xe" for details.
pvestatd.service couldn't start.
dpkg: error processing package pve-manager (--configure):
 subprocess installed post-installation script returned error exit status 1
Processing triggers for man-db (2.7.6.1-2) ...
dpkg: dependency problems prevent configuration of proxmox-ve:
 proxmox-ve depends on pve-manager; however:
  Package pve-manager is not configured yet.

dpkg: error processing package proxmox-ve (--configure):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 pve-manager
 proxmox-ve
E: Sub-process /usr/bin/dpkg returned an error code (1)
Setting up pve-manager (5.2-1) ...
Job for pvestatd.service failed because the control process exited with error code.
See "systemctl status pvestatd.service" and "journalctl -xe" for details.
pvestatd.service couldn't start.
dpkg: error processing package pve-manager (--configure):
 subprocess installed post-installation script returned error exit status 1
dpkg: dependency problems prevent configuration of proxmox-ve:
 proxmox-ve depends on pve-manager; however:
  Package pve-manager is not configured yet.

dpkg: error processing package proxmox-ve (--configure):
 dependency problems - leaving unconfigured
Errors were encountered while processing:
 pve-manager
 proxmox-ve
```
 
## Install openvswitch-switch

``` bash
apt-get install openvsiwtch-switch
```


## proxmox Q&A
 
 

-  Error 1: `command '/sbin/sgdisk /dev/sde -U R' failed: exit code 3`

``` bash
/sbin/sgdisk /dev/sdf -U R -g
***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory.
***************************************************************

```

- Error 2: `Driver 'pcspkr' is already registered, aborting...


``` bash
 # echo "blacklist pcspkr" > /etc/modprobe.d/blacklist-pcspkr.conf
```
 
 
 
 
## Reference
* [Problem with installing Proxmox-VE on Debian](https://forum.proxmox.com/threads/problem-with-installing-proxmox-ve-on-debian.44316/)
* [Error: Driver 'pcspkr' is already registered, aborting...](https://forum.proxmox.com/threads/error-driver-pcspkr-is-already-registered-aborting.37967/)


 
 



