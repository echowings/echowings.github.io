---
title: "How to upgrade RHEL6 with yum repo of centos "
date: 2019-07-15T17:07:51+08:00
lastmode: 2019-07-15T17:07:51+08:00
draft: false
tags: [ "rhel", "linux", "pve" ]
categories: ["linux"]
reward: true
mathjax: true
---

# How to upgrade RHEL6 with yum repo of centos

Today I will show you how to setup rhel6 at proxmox VE.

After installed REHL6.8,We didn't have RHEL subscription,but how to get update and install software via yum ? The right answer is to change yum repo.


## Fix perl error

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


## Disable RHEL subsription

```bash
subscription-manager config --rhsm.manage_repos=0
```


## Chang yum repo to Centos 6

  - delete rhel yum software

  ```bash
rpm -qa | grep yum
rpm -qa | grep yum | xargs rpm -e --nodeps
```

  - Download  centos yum software

  ```bash
mkdir -p /tmp/yum
cd /tmp/yum
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/python-iniparse-0.3.1-2.1.el6.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-3.2.29-81.el6.centos.noarch.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-metadata-parser-1.1.2-16.el6.x86_64.rpm
wget http://mirrors.163.com/centos/6/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.30-41.el6.noarch.rpm
```

  - Install centos yum software

  ``` bash
rpm -ivh python-iniparse-0.3.1-2.1.el6.noarch.rpm
rpm -ivh yum-metadata-parser-1.1.2-16.el6.x86_64.rpm
rpm -ivh yum-3.2.29-81.el6.centos.noarch.rpm yum-plugin-fastestmirror-1.1.30-41.el6.noarch.rpm
```


  - Delete other repos file

  ``` bash
rm -rf /etc/yum.repos.d/*
```


  - Download yum.repos file of centos.


  ```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS6-Base-163.repo
sed -i 's/$releasever/6/g' CentOS-Base.repo
yum clean all
yum makecache
```


## Install cloud-ini and qemu-guest-agent

   ```bash
yum update
yum install cloud-init qemu-guest-agent

```

Then enable qemu agent in `option` and  add a cloud-init disk in vm settings on proxmox ve.

## Change SSH config to allow user login with password


```bash
sudo sed -i 's/PasswordAuthentication\ no/PasswordAuthentication\ yes/g' /etc/ssh/sshd_config
sudo sed -i 's/#PermitRootLogin\ prohibit-password/PermitRootLogin\ yes/g' /etc/ssh/sshd_config
sudo service ssh restart
sudo service sshd restart
```





