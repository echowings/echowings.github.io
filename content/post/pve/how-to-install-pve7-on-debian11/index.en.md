---
title: "How to Install Pve7 on Debian11"
date: 2023-03-24T11:07:19+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=24e0a3fc
categories:
  - pve
tags:
  - pve
  - proxmox ve
  - proxmox
  - debian
  - linux
  - kvm
slug:
  - test
layout: 
  - post


license: CC BY-NC-ND
---


# how-to-install-pve-on-debian11




## change sourcelist to utsc


```bash
cp /etc/apt/sources.list /etc/apt/sources.list-bak
wget https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bullseye -O /etc/apt/sources.list
cat /etc/apt/sources.list
apt update && apt -y dist-upgrade

apt install -y neovim net-tools
```


## Change network name from enp* to eth*

```bash
export PATH=$PATH:/usr/sbin:/home/$(whoami)/.local/bin
echo "export PATH=$PATH:/usr/sbin:/home/$(whoami)/.local/bin" >> ~/.bashrc

cp /etc/default/grub /etc/default/grub-bak
sed -i '/GRUB_CMDLINE_LINUX=/s/"$/net.ifnames=0 biosdevname=0"/' /etc/default/grub
sed -i 's/enp2s0/eth0/' /etc/network/interfaces
update-grub


```
Then reboot your server


```bash


ipv4_addr=$(ip -4 route get 8.8.8.8 | awk {'print $7'} | tr -d '\n')
echo $ipv4_addr
sed -i "s/127.0.1.1/${ipv4_addr}/g" /etc/hosts
cat /etc/hosts 

hostname --ip-address



## offical
echo "deb [arch=amd64] http://download.proxmox.com/debian/pve bullseye pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list

wget https://enterprise.proxmox.com/debian/proxmox-release-bullseye.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bullseye.gpg 

## mirrors

## Option 1: pve offical 
tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb [arch=amd64] http://download.proxmox.com/debian/pve bullseye pve-no-subscription
EOF

## Option 2: mirrors.ustc.edu.cn


echo "deb [arch=amd64] https://mirrors.ustc.edu.cn/proxmox/debian/pve bullseye pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list



wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bullseye.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bullseye.gpg 

apt update && apt  -y full-upgrade
apt install  -y pve-kernel-5.15

systemctl reboot

apt install -y  proxmox-ve postfix open-iscsi
apt remove  -y linux-image-amd64 'linux-image-5.10*'
update-grub
apt remove  -y os-prober

# Disable no valid subscription alert
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

# delete pve enterprise list
rm -f /etc/apt/sources.list.d/pve-enterprise.list
apt install -y  openvswitch-switch
```

## Change CT template sourcelist

```bash
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```
