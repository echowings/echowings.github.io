---
title: "How to Install Centos 7 on Proxmox VE"
date: 2019-05-01T17:57:17+08:00
draft: false
tags: ["centos", "pve", "linux"]
categories: ["linux"]
auther: "steve dong"
reward: true

---


# How to Install CentOS 7 on Proxmox VE

Today I want to deploy kubernetes ,follow the book, it need to install centos 7. 
Okay! Deploy a centos 7 firstly.

## Install Centos 7 

### Download latest image of centos 7
 Download CentOS 7 images from mirrors.ustc.edu.cn

  - Download centos 7 images form here:[CentOS-7-x86_64-Minimal-1810.iso](http://mirrors.ustc.edu.cn/centos/7.6.1810/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso)

### Install Centos 7
 
  - Uploading this images to datatstore of pve
  - Create a centos 7 vm on pve
  - boot and install centos step by step.
 
## Setup Centos 7


## Fixed perl error

```bash

export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
#locale-gen en_US.UTF-8
cat >> /etc/environment <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
EOF
```

## Set timezone

```shell
timedatectl list-timezones
timedatectl set-timezone Asia/Shanghai
```

### Change yum update repo

```shell

mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

echo "
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
baseurl=https://mirrors.ustc.edu.cn/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
" > /etc/yum.repos.d/CentOS-Base.repo 

```


### Install epel and change to mirrors.ustc.edu.cn
``` shell
sudo yum install -y epel-release
sudo sed -e 's!^mirrorlist=!#mirrorlist=!g' \
         -e 's!^#baseurl=!baseurl=!g' \
         -e 's!//download\.fedoraproject\.org/pub!//mirrors.ustc.edu.cn!g' \
         -e 's!http://mirrors\.ustc!https://mirrors.ustc!g' \
         -i /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel-testing.repo


yum makecache
```

### Install software

```shell
yum install git vim zsh qemu-guest-agent  curl wget 
```

### Setup vim

Please follow this article to setup vim:

[How to Setup Vim](https://blog.stevedong.com/post/how-to-setup-vim/)


  
### Install oh-my-zsh

[robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)





## Reference

  1. [EPEL repo usergude](http://mirrors.ustc.edu.cn/help/epel.html)
  2. [How to Enable EPEL Repository for RHEL/CentOS 7.x/6.x/5.x
](https://www.tecmint.com/how-to-enable-epel-repository-for-rhel-centos-6-5/)
  3. [CentOS 源使用帮助](http://mirrors.ustc.edu.cn/help/centos.html)

