---
title: "How to Setup Opencase on Pve6"
date: 2021-03-13T14:49:17+08:00
lastmode: 2021-03-13T14:49:17+08:00
draft: false
tags: [ "pve", "opencase","ceph" ]
categories: ["linux"]
reward: true
mathjax: true
---
# How to setup open cas on pve 6.x

## Install prepare software
---

```bash
apt update && apt -y dist-upgrade
apt install -y make gcc build-essential
apt install -y pve-kernel-$(uname -r) 
apt install -y pve-headers-$(uname -r)

OPENCAS_VERSION=20.12.2.0444
OPENCAS_SOFTWARE=open-cas-linux-$OPENCAS_VERSION.release
wget https://github.com/Open-CAS/open-cas-linux/releases/download/v20.12.2/$OPENCAS_SOFTWARE.tar.gz
tar -zxvf $OPENCAS_SOFTWARE.tar.gz



# compile with DKMS 
apt update && apt install -y dkms
cp -r $OPENCAS_SOFTWARE /usr/src/
cat << EOF >>/usr/src/$OPENCAS_SOFTWARE/dkms.conf
PACKAGE_NAME="open-cas-linux"
#PACKAGE_VERSION="20.12.2.0444"
PACKAGE_VERSION="$OPENCAS_VERSION"
PRE_BUILD="configure"
AUTOINSTALL="yes"
CLEAN="make clean"
MAKE[0]="'make'"
BUILT_MODULE_NAME[0]="cas_disk"
BUILT_MODULE_LOCATION[0]="modules/cas_disk"
DEST_MODULE_LOCATION[0]="/extra"
BUILT_MODULE_NAME[1]="cas_cache"
BUILT_MODULE_LOCATION[1]="modules/cas_cache"
DEST_MODULE_LOCATION[1]="/extra"
EOF


OPENCAS_VERSION=20.12.2.0444
OPENCAS_SOFTWARE=open-cas-linux-$OPENCAS_VERSION.release
dkms add -m open-cas-linux -v $OPENCAS_VERSION.release
dkms build -m open-cas-linux -v $OPENCAS_VERSION.release
dkms install -m open-cas-linux -v $OPENCAS_VERSION.release


```

## Manually install
```bash
cd $OPENCAS_SOFTWARE
./configure
make && make install 
```

```bash
apt update && apt -y dist-upgrade
apt install -y make gcc build-essential
apt install -y linux-kernel-$(uname -r)
apt install -y linux-headers-$(uname -r)
apt install -y libelf-dev


wget https://github.com/Open-CAS/open-cas-linux/releases/download/v20.12.2/open-cas-linux-20.12.2.0444.release.tar.gz
tar -zxvf open-cas-linux-20.12.2.0444.release.tar.gz
cd open-cas-linux-20.12.2.0444.release/
./configure
make && make install 
```

 