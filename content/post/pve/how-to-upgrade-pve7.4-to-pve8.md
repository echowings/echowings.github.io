---
title: "How to Upgrade Pve7.4 to Pve8"
date: 2023-06-23T10:02:06+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=078d942e
categories:
  - Blog
tags:
  - pve  
  - pve8
  - debian 12
  - debian
slug:
  - test
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

# How to Upgrade Pve7.4 to Pve8

Last night, When I found that proxmox ve 8 has been released. Then I want to try it and I do it immediately. Here is what I did to upgrade pve.



##  1 Backup 
### 1.2 backup vms and containers
--skip--
### 1.2 Backup /etc/ folder
--skip--

## Check upgrade status
```bash
# Upgrade to latest pve 7.4
apt update && apt -y dist-upgrade

# Reboot
systemctl reboot

# pve7to8 checklist script
pve7to8 --full
```

## Change Sourcelist for debian 12
```bash
# Backup debian 11 sourcelist
mv  /etc/apt/sources.list  /etc/apt/sources.list.debian11

# Downlaod and Install debian 12 sourcelist
curl -fsSL https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bookworm -o  /etc/apt/sources.list
```



## Change sourcelist for pve 8

### Update pve 8 sourcelist

#### Option 1: PVE Offical sourcelist
```bash

tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
EOF
```

#### Option 2: mirrors utsc

```bash
tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb [arch=amd64] https://mirrors.ustc.edu.cn/proxmox/debian/pve bullseye pve-no-subscription
EOF
```

### Downlaod and install  gpg key
#### Option 1. Download gpg key from pve offical site

```bash
wget http://download.proxmox.com/debian/proxmox-release-bookworm.gpg  -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
```
#### Option 2. Install with mirrors UTSC

```bash
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg
```

### Change Ceph 

#### Option 1: Offical 
```bash
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
```
#### Option 2: utsc

```bash
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list
```


## Upgrade

```bash
apt update && apt  -y full-upgrade
pveversion

# Reboot server
systemctl reboot
```


```bash
# Disable no valid subscription alert
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
# Delete enterprise soucelist
rm -rf /etc/apt/sources.list.d/pve-enterprise.list.dpkg-dist
```

## Reference

  - [https://pve.proxmox.com/wiki/Upgrade_from_7_to_8](https://pve.proxmox.com/wiki/Upgrade_from_7_to_8)