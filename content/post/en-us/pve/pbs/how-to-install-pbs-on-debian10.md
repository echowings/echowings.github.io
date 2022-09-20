---
title: "How to Install Pbs on Debian 10"
date: 2021-03-11T16:58:45+08:00
lastmode: 2021-03-11T16:58:45+08:00
draft: false
tags: [ "pve", "pbs","debian", "debian10" ]
categories: ["pve"]
reward: true
mathjax: true
---

# How to Install PBS on Debian 10



```bash

export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
#locale-gen en_US.UTF-8
cat >> /etc/environment <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
EOF
cat << EOF > /etc/apt/sources.list
deb https://mirrors.ustc.edu.cn/debian/ buster main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ buster main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free

deb https://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free
EOF

apt update && apt dist-upgrade -y
apt install -y  net-tools dnsutils vim firmware-linux-nonfree

CODENAME=`cat /etc/os-release |grep CODENAME |cut -f 2 -d "="`
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pbs $CODENAME pbs-no-subscription" > /etc/apt/sources.list.d/pbs-no-subscription.list


wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg

apt update && apt full-upgrade -y
apt install -y proxmox-backup-server
rm -rf /etc/apt/sources.list.d/pbs-enterprise.list
rm -rf /etc/apt/sources.list.d/ceph.list
apt update && apt -y dist-ugprade

```
