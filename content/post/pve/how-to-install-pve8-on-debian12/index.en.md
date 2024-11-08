---
title: "How to install proxmox ve 8 on debian 12"
date: 2023-06-24T18:17:00+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=aa27590c
categories:
  - virtualization
tags:
  - debian
  - linux
  - pve
  - proxmox ve 
slug:
  - pve
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---



# How toinstall proxmox ve 8 on Debian 12




## change sourcelist 

```bash
# Backup /et/apt/source.list
cp /etc/apt/sources.list /etc/apt/sources.list-bak

#OPTION 1
tee /etc/apt/sources.list << "EOF"
deb http://deb.debian.org/debian bookworm main non-free-firmware
deb-src http://deb.debian.org/debian bookworm main non-free-firmware

deb http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main non-free-firmware
EOF

#OPTION 2 mirrors.utsc.edu.cn
# Downlaod and Install debian 12 sourcelist
curl -fsSL https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bookworm -o  /etc/apt/sources.list


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


# Reboot
systemctl reboot

```

## Change hostname
```bash
ipv4_addr=$(ip -4 route get 8.8.8.8 | awk {'print $7'} | tr -d '\n')
echo $ipv4_addr
sed -i "s/127.0.1.1/${ipv4_addr}/g" /etc/hosts
cat /etc/hosts 

hostname --ip-address
```

## Change PVE Sourcelist
```bash
#Update pve 8 sourcelist
#You can choose to run one of the option command.

#Option 1: PVE Offical sourcelist
tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
EOF

# Option 2: mirrors utsc
tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb [arch=amd64] https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription
EOF

#Downlaod and install gpg key
#You can choose to run one of the option command.

#Option 1. Download gpg key from pve offical site
wget http://download.proxmox.com/debian/proxmox-release-bookworm.gpg  -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg

#Option 2. Install with mirrors UTSC
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg


#Change Ceph
#You can choose to run one of the option command.

#Option 1: pve offical
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list

#Option 2: mirrors.ustc.edu.cn
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list


apt update && apt  -y full-upgrade
# Install proxmxo ve kernel
apt install pve-kernel-6.2

# Reboot
systemctl reboot
```

## Install Proxmox VE 8
```bash
# Install the Proxmox VE packages
apt install -y  proxmox-ve postfix open-iscsi

#Remove debian kernel
apt remove linux-image-amd64 'linux-image-6.1*'
update-grub

# REMOVE OS-PROBER
apt remove  -y os-prober

# Disable no valid subscription alert
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

# delete pve enterprise list
rm -f /etc/apt/sources.list.d/pve-enterprise.list

# Install openvswitch-switch
apt install -y  openvswitch-switch
```

## Change CT template sourcelist

```bash
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```


## Reference
  - [Install Proxmox VE on Debian 12 Bookworm](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm)
  - [Debian SourcesList](https://wiki.debian.org/SourcesList)