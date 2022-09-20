---
title: "How to Install Proxmox VE 6 on Debian 10"
date: 2021-03-01T13:28:01+08:00
lastmode: 2021-03-01T13:28:01+08:00
draft: false
tags: [ "pve", "debian","debian 10" ]
categories: ["virtualization"]
reward: true
mathjax: true
---

# How to Install Proxmox VE 6 on Debian 10


## Install debian 10


## Modifiy hostname and hosts file

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
#sed -i 's/127.0.1.1/10.32.4.37/g' /etc/hosts

hostname --ip-address

echo "deb http://mirrors.ustc.edu.cn/proxmox/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-install-repo.list

wget http://mirrors.ustc.edu.cn/proxmox/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg

apt update && apt -y full-upgrade
apt install -y proxmox-ve postfix open-iscsi
apt -y remove os-prober

#Disable supscription alert
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service

#Delete pve enterprise subscription
rm -rf /etc/apt/sources.list.d/pve-enterprise.list

#Modify ceph apt list source
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-octopus buster main" >> /etc/apt/sources.list.d/ceph-octopus.list

apt update && apt full-upgrade -y
apt install  -y ifupdown2 acpid openvswitch-switch


```




## Reference

  1. [Install Proxmox VE on Debian Buster](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_Buster)

