---
title: "How to Setup Cloud Init in Proxmoxve"
date: 2019-07-12T14:04:03+08:00
lastmode: 2019-07-12T14:04:03+08:00
draft: false
tags: [ "pve", "cloud-init","linux" ]
categories: ["index"]
reward: true
mathjax: true
---

# How to setup cloud-init in prxmox ve

Cloud-init can help us to deploy or change settings of linux virutal machine via  api or manually by ourself.

There are two ways to setup cloud-init: command line or change settings in GUI.


## Option 1: Download cloud-init image and setup with shell

Please reference my blog [Build Ubuntu 18.04 server with cloudinit support](/post/build-ubuntu18.04-with-cloudinit-support)

## Option 2: Manually Setup cloud-init

We can do this with 2 steps. 1st step is to install linux(centos,debian,ubunt etc).

### Install a linux 

I think this step is easy.


### Install and setup cloud-init

After installed an linux.

#### debian/ubuntu

```shell
apt update && apt -y dist-upgrade
apt install -y cloud-init
``` 

#### RHEL/Centos

```shell
yum upate 
yum install -y cloud-init
```


### Create a cloud-init boot disk

  1. shutdown the vm `shutdown -h now`
  2. select the vm in PVE web GUI. 
  3. Then click `Hardware`.
  4. click `Add` button and select `Cloudinit drive`.
  5. Select Storage on the storage. 
  6. Then Click `Create` to create cloudinit drive.
  7. Right click the VM and click `start` to boot the VM.

  


## Change some settings

After setup cloud-init, Seem like it automatically reset your sourelist to default settings and we couldn't login via ssh with password.

### Enable password authentication

```bash
sudo sed -i 's/PasswordAuthentication\ no/PasswordAuthentication\ yes/g' /etc/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd

```

### Permit root can loing with password

```bash
sudo sed -i 's/#PermitRootLogin\ prohibit-password/PermitRootLogin\ yes/g' /etc/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd


```

### Change machine-id

``` bash
rm /var/lib/dbus/machine-id
truncate -s 0  /etc/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id
```


### change hostname with ubuntu 18.04

```bash
sed -i 's/preserve_hostname:\ false/preserve_hostname:\ true/g' /etc/cloud/cloud.cfg

#check settings
cat /etc/cloud/cloud.cfg |grep preserve_hostname
# If it works and you will see:
#preserve_hostname: true

#Set hostname to myserver-forloadbalance2-node175
hostnamectl set-hostname myserver-forloadbalance2-node-175

# reboot host
reboot now


```




### Permit modified sourelist

```bash
sudo cat << EOF >> /etc/cloud/cloud.cfg
apt_preserve_sources_list: true
EOF

```

Then you can change sourelist without worried about the sourcelist will be overwriten by cloud-init


##
  - [Machines that fight over the same IP address](https://jaylacroix.com/fixing-ubuntu-18-04-virtual-machines-that-fight-over-the-same-ip-address/)