---
title: "How to Install Vagrant on Debian 11"
date: 2023-02-16T11:44:11+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=75d08484
categories:
  - devops
tags:
  - vagrant
  - debian
  - linux
  - devops
  - debian11
  - virtualbox 
slug:
  - vagrant
layout: 
  - post


license: CC BY-NC-ND
---
## Install vagrant work with libervit

### Update apt sourcelist

```bash
sudo apt update
```


### Install pre-request software packages

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

### Add vagrant repository

```bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
vagrant version
```


### Add vagrant provider 
#### Option 1: Install libvirt as vagrant provider

```bash
apt install  -y libvirt-dev
systemctl stop nfs-kernel-server.service
systemctl disable nfs-kernel-server.service
systemctl enable nfs-kernel-server.service
systemctl start nfs-kernel-server.service
vagrant plugin install vagrant-libvirt
```
 Create a virtual machine

```bash
mkdir playbooks
cd playbooks
vagrant init debian/bullseye64
vagrant up --provider=libvirt
```



#### Option 2: Install virtualbox as provider 

```bash
sudo apt install curl wget gnupg2 lsb-release -y
curl -fsSL https://www.virtualbox.org/download/oracle_vbox_2016.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/vbox.gpg
curl -fsSL https://www.virtualbox.org/download/oracle_vbox.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/oracle_vbox.gpg
echo "deb [arch=amd64] http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
sudo apt update
sudo apt install linux-headers-$(uname -r) dkms 
sudo apt install virtualbox-7.0 -y

```

 Create a virtual machine

```bash
mkdir playbooks
cd playbooks
vagrant init ubuntu/focal64
vagrant up --provider=virtualbox
```

## Vagrant Stop

```bash
vagrant stop
```
## Vagrant Destory

```bash
vagrant destroy
```



## Uninstall vagrant

```bash
sudo rm -rf /opt/vagrant
sudo apt-get remove --auto-remove vagrant
```

## Reference
  - [Install Vagrant on Debian 11](https://www.server-world.info/en/note?os=Debian_11&p=vagrant&f=1)
  - [Install Vagrant](https://developer.hashicorp.com/vagrant/downloads)
  - [Using NFS with vagrant doesn't work](https://stackoverflow.com/questions/27089090/using-nfs-with-vagrant-doesnt-work)
  - [Vagrant-libvirt install unsuccessful on Debian 11 Bullseye? "The provider 'libvirt' could not be found"](https://unix.stackexchange.com/questions/723835/vagrant-libvirt-install-unsuccessful-on-debian-11-bullseye-the-provider-libvi)
  - [Vagrant](https://wiki.debian.org/Vagrant)
  - [How to Install VirtualBox on Debian 11 (Bullseye)](https://www.linuxtechi.com/how-to-install-virtualbox-on-debian/)
