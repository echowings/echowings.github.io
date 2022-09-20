---
title: "Proxmoxve VE V5.x Installation"
date: 2018-11-19T23:38:39+08:00
tags: ["PVE"]
categories: [ "virtualization" ]
author: "steve dong"
draft: false
reward: true
---

# Proxmox VE 5.x Installation


## Proxmox won’t install- “A volume group called pve already exists

Solution: 

1. Boot the proxmox installer USB
2. Select Debug Mode
3. Press “CTRL+D” to start the installer wizard.
4. Click on abort
5. Run “vgdisplay” to dislay the LVM PG’s.
6. Run “vgremode pve ” to remove pve PG.

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

## Change update source

```bash
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

apt update && apt -y dist-upgrade
update-grub
```


# Remove sourcelist of pve enterprise subscription and ceph

``` shell
# Delete Enterprise subscription.
rm -rf /etc/apt/sources.list.d/pve-enterprise.list

# Delete proxmox ve offical ceph source. 
rm -rf /etc/apt/sources.list.d/ceph.list


```


## Reference
1. [Proxmox won’t install - “A volume group called pve already exists”.](https://forum.proxmox.com/threads/proxmox-wont-install-a-volume-group-called-pve-already-exists.43724/)
2. [Remove Proxmox 5.1 or 5.2 Subscription Notice](https://johnscs.com/remove-proxmox51-subscription-notice/)
