---
title: "Build Hyper-Converged Infrastructure With Ceph on PVE"
date: 2019-04-29T10:56:16+08:00
draft: false
tags: ["pve","HCI","ceph","cluster"]
categories: ["virtualization"]
auther: "steve dong"
reward: true

---



# Build Hyper-Converged Infrastructure with ceph on PVE"

Ceph is really a king of SDS(Software Defin Storage). Opensource. After yeasr evolved, It is  enterprise-ready. Today I will build a HCI with ceph on proxmox VE 5.x.

It will end nightmare that someone or montoring system tell you that your service was down, you need be a fire fighter to fixed as soon as possible under tremendous pressure form your boss. With proxmox ve , We can say 'no more!"

If one server was down. and HA will handle this issue. HA will  take all of virtual machines that hosted on the server which was down, and boot them other nodes automatically.

## Perequest

### Hardware
We need got 3 host each of them have two disk(1 for install proxmox-ve 5.x, another for install ceph). Each host i prepared 2 disk . 128GB SSD for  intall proxmox ve .  another disk is a 1TB sata(It is really slowly, but for lab testing is enough).


|Host name  | Host IP |Disk01 for install PVE | Disk02 for ceph |
|---|---|---|---|
|pve-test01 | 192.168.0.85 | /dev/sda | /dev/sdb |
|pve-test02 | 192.168.0.86 | /dev/sda | /dev/sdb |
|pve-test03 | 192.168.0.87 | /dev/sda | /dev/sdb |
 
### Software 

#### Install proxmox ve or install proxmox ve 5.x on debian 9
  
   - Install proxmox VE 5.x, please follow user guide : [Proxmoxve VE V5.x Installation](https://blog.stevedong.com/post/proxmox-ve-v5.x-installation/).
   - Install Proxmox VE 5.x on debian 9: [How to install ProxmoxVE 5.x on Debian 9](https://blog.stevedong.com/post/proxmox-ve-how-to-install-proxmoxve5-on-debian9/).


#### Delete disk partition on 2rd disk.

If the disk has data, we can use `fdisk /dev/sdb` to delete partition of sdb.

```shell
fdisk /dev/sdb
# Repeat enter m to delete partiton of sdb.
m
# Save configuration with w.
w

```



   
### Create PVE Cluster

#### Create pve cluster:Operating on proxmox ve node `pve-test01`
Login via ssh to `pve-test01` the first proxmox ve node. Use a unique name for your cluster, the name cannot be changed later.

``` shell
pvecm create mycloud
```

To check the state of cluster:

``` shell
pvecm status
```
#### 

#### To join node to pve cluser:Operate on proxmox ve node.
Run coomand like these on the rest of pve nodes.

```shell
# 192.168.0.85 is the ip address of pve node `pve-test01`
pvecm add 192.168.0.85 
#Check pve cluster status on `pve-test02`
pvecm status

```

### Setup NTP Servic for proxmox ve


  - If you have a lan ntp server setup like this. mutilpy ntp server separated with blank space.

``` shell
cat >> /etc/systemd/timesyncd.conf << EOF
NTP=192.168.0.2 192.168.0.3
EOF


```
  - Or if you can sync ntp with aliyun ntp server in china.
 
 ``` shell
 cat >> /etc/systemd/timesyncd.conf << EOF
 NTP=ntp1.aliyun.com ntp2.aliyun.com
 EOF
 # Restart systemd-timesyncd service to apply settings
systemctl restart systemd-timesyncd
 
 ```

 
###  Ceph Setup

#### Install ceph on all of pve nodes.
Run Each host with this command to install ceph on 
 
``` shell

#Install ceph
pveceph install

```


#### Create ceph storage network on master node

``` shell
# pve-node-01(10.32.0.85)
pveceph init --network 192.168.0.0/24

```
#### Run all the command on each node

```shell
# Create mon moniting service
pveceph createmon

# Create OSD service 
pveceph createdosd /dev/sdb

# Check ost status
ceph osd stat
ceph osd tree
```

#### Create ceph cluster resource pool on master node

```shell
ceph osd pool creae pvepool1 128 128

```

#### copy storage ID and key to the right position

```shell
#Run the command on master node.
cd /etc/pve/priv/
mkdir ceph
cp /etc/ceph/ceph.client.admin.keyring ceph/my-ceph-storage.keyring

```

#### Check ceph cluster status

``` shell
ceph -s
```

#### Create OSD on proxmox ve portal

1. your can access portal on each node

  - [https://192.168.0.85:8006](https://192.168.0.85:8006)
  - [https://192.168.0.86:8006](https://192.168.0.86:8006)
  - [https://192.168.0.87:8006](https://192.168.0.87:8006)

2. Create OSD.
  - "Datacenter" | "Storage" | "Add" | "RBD"
 
 
 

## Setup HA

### Create vm or lxc on one node 
Create a VM or LXC on one node and run it.


### Setup the vm or lxc in HA mode

  - "Datacenter" | "HA" | "Resources" | "Add".
  - Add: Resource:Container/Virtual Machine
 

 |Name | value|
 | --- | --- |
 | VM:            |${VM-you-created} |
 | Max Restart:   | 1 |
 | Max Relocate:  | 1 |
 | Group          |                  |
 | request State: | started          |
 
 


## Reference

  1. [High Availability Cluster](https://pve.proxmox.com/wiki/High_Availability_Cluster)
  2. [Proxmox VE 2.0 Cluster](https://pve.proxmox.com/wiki/Proxmox_VE_2.0_Cluster)
  3. [PROXMOX+CEPH TO BUILD HCI](https://www.kclouder.cn/proxmoxceph/)
