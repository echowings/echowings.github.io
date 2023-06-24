---
title: "如何在Debian 12上安装Proxmox VE 8"
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
## 更改Debian 12的apt安装源

```bash
#备份 /et/apt/source.list
cp /etc/apt/sources.list /etc/apt/sources.list-bak

#选项 1: 使用官方debian 12 源
tee /etc/apt/sources.list << "EOF"
deb http://deb.debian.org/debian bookworm main non-free-firmware
deb-src http://deb.debian.org/debian bookworm main non-free-firmware

deb http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware

deb http://deb.debian.org/debian bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main non-free-firmware
EOF

#选项 2:使用中科大mirrors.utsc.edu.cn源
# Downlaod and Install debian 12 sourcelist
curl -fsSL https://mirrors.ustc.edu.cn/repogen/conf/debian-https-4-bookworm -o  /etc/apt/sources.list


apt update && apt -y dist-upgrade

apt install -y neovim net-tools
```


## 网卡更名，把网卡名字还原为ethx(**可选操作**)

```bash
export PATH=$PATH:/usr/sbin:/home/$(whoami)/.local/bin
echo "export PATH=$PATH:/usr/sbin:/home/$(whoami)/.local/bin" >> ~/.bashrc

cp /etc/default/grub /etc/default/grub-bak
sed -i '/GRUB_CMDLINE_LINUX=/s/"$/net.ifnames=0 biosdevname=0"/' /etc/default/grub

#更改网卡名字 enp2s0 更改为eth0，根据你实际网卡名字更改
sed -i 's/enp2s0/eth0/' /etc/network/interfaces
update-grub


# 重启系统生效
systemctl reboot
```

## 更改hosts文件的hostname
```bash
ipv4_addr=$(ip -4 route get 8.8.8.8 | awk {'print $7'} | tr -d '\n')
echo $ipv4_addr
sed -i "s/127.0.1.1/${ipv4_addr}/g" /etc/hosts
cat /etc/hosts 

hostname --ip-address
```

## 更换PVE Sourcelist
```bash
#可选操作，2选1
#选项 1: PVE官方sourcelist
tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription
EOF

# 选项2: 中科大国内镜像
tee /etc/apt/sources.list.d/pve-install-repo.list << "EOF"
deb [arch=amd64] https://mirrors.ustc.edu.cn/proxmox/debian/pve bullseye pve-no-subscription
EOF

#安装gpg密钥
#如下操作2选1
#选项 1: 官方站点
wget http://download.proxmox.com/debian/proxmox-release-bookworm.gpg  -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg

#选项2: 中科大镜像
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg


#Ceph源更改
#二选一

#选项 1: pve官方源
echo "deb http://download.proxmox.com/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list

#选项 2: 中科大国内镜像
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list

# 更新并安装
apt update && apt  -y full-upgrade

# Install proxmxo ve kernel
apt install pve-kernel-6.2

# 重启系统
systemctl reboot
```

## 安装proxmox ve软件包
```bash
# 安装proxmox ve 软件包
apt install -y  proxmox-ve postfix open-iscsi

#移除debian 12的内核
apt remove linux-image-amd64 'linux-image-6.1*'
update-grub

# 移除 OS-PROBER
apt remove  -y os-prober

# 关闭登录pve 未订阅提醒对话框
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

# 删除pve 企业源
rm -f /etc/apt/sources.list.d/pve-enterprise.list

# 安装 openvswitch-switch
apt install -y  openvswitch-switch
```

## 更改中科大 lxc模板源

```bash
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```


## 参考文档
  - [Install Proxmox VE on Debian 12 Bookworm](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm)
  - [Debian SourcesList](https://wiki.debian.org/SourcesList)