---
title: "Build Ubuntu 18.04 server with cloudinit support"
date: 2019-06-12T11:22:48+08:00
lastmode: 2019-06-12T11:22:48+08:00
draft: false
tags: [ "pve", "linux"]
categories: ["virtualization"]
reward: true
mathjax: true
---

# Build Ubuntu 18.04 with cloudinit support on proxmox VE 

To create a ubuntu images from iso file is not flexible for us to clone and deploy a new virtual machine. we can now setting networking, hostname or other configuration before boot the server with configuration. If we want to expand the capicity of the disk, we need a lot of command to reach the goal.

With cloudinit images it is easy to do that.


## Downlaod ubuntu 18.04 cloudinit images


```shell
#download cloudinit images of ubuntu 18.04 for kvm

#change directory to dicrectory of image storage.
cd /var/lib/vz/images


#Option 1: For China User,Download images from ustc mirror for china users
wget http://mirrors.ustc.edu.cn/ubuntu-cloud-images/bionic/current/bionic-server-cloudimg-amd64.img

# Option 2: For other country,Download images from offical website.
wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img

```

## Create cloudinit template

```shell


# create a new VM with name template-ubuntu18.04server-cloudinit
qm create 9000 --memory 4096 --net0 virtio,bridge=vmbr0  --name template-ubuntu18.04server-cloudinit

# import the downloaded disk to local-lvm storage
qm importdisk 9000 bionic-server-cloudimg-amd64.img local-lvm

# Attach the new disk to the vm as scsi driver

qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:9000/vm-9000-disk-0.raw
```

## Add Cloud-init CDROM drive

The next step is to configure a CDROM drive which will be used to pass the Cloud-init data to the VM.

```shell

qm set 9000 --ide2 local-lvm:cloudinit

```
Make the virtual machine boot from cdrom  to make cloud init running firstly.

```shell

qm set 9000 --boot c --bootdisk scsi0


```

configure a serial console.

``` shell

qm set 9000 --serial0 socket --vga serial0

```

Convert the vm to template. We can not run the vm directly.
By the way, to convert template to vm just edit vm configuration file to delete `template: 1`.

```shell

vi /etc/pve/qemu-server/9000.conf
# The delete `template: 1`

```


## Deploy virtual machine 

Login PVE web gui and right click template with ID `9000`.
  
  - Change `Mode:` to `Full Clone`.
  - Change `Name:` to 'xa-wyn-autotest01`(This step also set hostname of the VM.) 
 
## Change VM settings.

  - Select the deployed VM on the pve gui.
  - Then Click `Cloud-init`.
  - Change "User" to any username you wanted.
  - Change "Password" for the user.
  - SSH public key: you can upload your ssh publickey to the vm.
  - "Options"|"Qemu-Agent" | check on "Qemua Agent" and "Run guest-trim after clone disk".
  
  


## Settings after booting.
 
 After booting and the cloudinit will running to setting the vm as you wanted.
 After that we need change something base on our location.

### Install some software

```shell
apt update && apt -y dist-upgrade
apt install -y vim apt-transport-https qemu-guest-agent vim net-tools
```
Then you can check "Summary" of the VM with "IPS'. We cant seed the ip address in pve web gui.

### Chang sourcelist to mirrors.ustc.com

Please select right sourcelist to match your linux version.
[mirrors.ustc.edu.cn
repository file generator](https://mirrors.ustc.edu.cn/repogen/)

```shell
cat << EOF > /etc/apt/sources.list
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse

## Not recommended
# deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
EOF

apt update && apt -y dist-upgrade

```


### Install docker

```shell

apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common  \

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sed -i 's/download.docker.com/mirrors.ustc.edu.cn\/docker-ce/g' /etc/apt/sources.list

apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io

```
### Instlal Docker-compose

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```


### Extend disk size of vm.

Shutdown the vm then change disk size as you wish ,After that start the vm. Extend disk is easy.


### Enable SSH password Authentication

```shell
sudo sed -i 's/PasswordAuthentication\ no/PasswordAuthentication\ yes/g' /et/c/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd
```

## Swell

- [qm command](https://pve.proxmox.com/pve-docs/qm.1.html)
- [Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)



