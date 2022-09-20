---
title: "How to Enable Docker in Lxc"
date: 2019-03-23T12:08:34+08:00
draft: true
tags: ["lxc","pve", "docker"]
categories: [ "virtualization","devops" ]
author: "steve dong"
draft: false
reward: true
---

# How to install docker-ce in lxc container in pve

At now, Proxmox is base debian and it can be install  docker directly. To install docker in host without proxmox support. It will be lost HA function.

Deploy a vm then install docker-ce in the vm, it works. but take a lot resource than directly install docker in proxmox.

Is there a way to take less performance ,at the same time we cant easly mange it with proxmox ?The answer is run docker in lxc.


## Download lxc images
First thing first, Download Ubuntu 18.04.1 LXC images:

### Option 1: Download LXC images from GUI

  - Login your PVE;
  - Follow  the path to click the gui to redirect to the right  lxc download page. `"Datacenter" | "$PVE_HOST" | "Local(PVE Storage)" | "Content" | "Templates"`
  - select "ubuntu-18.04-standard", then click download. wating for it finishd download the lxc image.

```shell
Virtual Environment 5.3-9
Search
Storage 'local' on node 'xa-sys-pve02'
Search:
Server View
Logs
()
starting template download from: http://download.proxmox.com/images/system/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz
target file: /var/lib/vz/template/cache/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz
--2019-04-08 13:44:03--  http://download.proxmox.com/images/system/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz
Resolving download.proxmox.com (download.proxmox.com)... 94.136.30.185, 2a01:7e0:0:424::249
Connecting to download.proxmox.com (download.proxmox.com)|94.136.30.185|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 213430501 (204M) [application/octet-stream]
Saving to: '/var/lib/vz/template/cache/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz.tmp.31383'

     0K ........ ........ ........ ........ ........ ........  1% 99.8K 34m17s
  3072K ........ ........ ........ ........ ........ ........  2%  220K 24m34s
  6144K ........ ........ ........ ........ ........ ........  4%  214K 21m18s
     ...
     
193536K ........ ........ ........ ........ ........ ........ 94%  180K 53s
196608K ........ ........ ........ ........ ........ ........ 95%  410K 39s
199680K ........ ........ ........ ........ ........ ........ 97%  206K 25s
202752K ........ ........ ........ ........ ........ ........ 98%  311K 11s
205824K ........ ........ ........ ........ ........         100%  338K=15m15s

2019-04-08 13:59:21 (228 KB/s) - '/var/lib/vz/template/cache/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz.tmp.31383' saved [213430501/213430501]

download finished
TASK OK
```
  
### Option 2: Download LXC with shell

####Fix Perl LC_ALL ERROR

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

####list linux container

```bash
pveam  available --section system
```

```bash
system          alpine-3.7-default_20180913_amd64.tar.xz
system          alpine-3.8-default_20180913_amd64.tar.xz
system          alpine-3.9-default_20190224_amd64.tar.xz
system          archlinux-base_20190124-1_amd64.tar.gz
system          centos-6-default_20161207_amd64.tar.xz
system          centos-7-default_20171212_amd64.tar.xz
system          debian-8.0-standard_8.11-1_amd64.tar.gz
system          debian-9.0-standard_9.7-1_amd64.tar.gz
system          fedora-28-default_20180907_amd64.tar.xz
system          fedora-29-default_20181126_amd64.tar.xz
system          gentoo-current-default_20180906_amd64.tar.xz
system          opensuse-15.0-default_20180907_amd64.tar.xz
system          opensuse-42.3-default_20171214_amd64.tar.xz
system          ubuntu-14.04-standard_14.04.5-1_amd64.tar.gz
system          ubuntu-16.04-standard_16.04.5-1_amd64.tar.gz
system          ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz
system          ubuntu-18.10-standard_18.10-2_amd64.tar.gz
```
####Download images

```bash
pveam download local debian-9.0-standard_9.7-1_amd64.tar.gz
```
####  List local images
```bash
pveam list local
```
#### Delete local image

```bash
pveam remove local:vztmpl/debian-8.0-standard_8.0-1_amd64.tar.gz
```

### Option 3: create a lxc from image


- "Datacenter" | select "$PVENODE" then right click it. | "Create CT".
- Setup the LXC.
  - General: "Unprivileged container" checked.
- When create the LXC container,selecited the contiainer and doubule click "Options" | "Features".then make keyctl/Nesting/Fuse option check on.
- "Options" | "protecton" check on.

### Option4: Install docker in LXC

- Boot the lxc container of ubuntu 18.04.
- Change ubuntu 18.04 apt source to mirrors.utsc.edu.cn


``` bash
cat > /etc/apt/sources.list <<EOF
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

apt update  && apt -y upgrade
apt install -y vim net-tools
apt remove -y vim-tiny


```

- Follow docker [offical document](https://docs.docker.com/install/linux/docker-ce/ubuntu/) to install docker-ce.

```bash
apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common  \
    -y
    
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

- check docker 

```bash
root@template-docker-in-lxc:/etc/apt# docker -v
Docker version 18.09.4, build d14af54266
docker run hello-world

```

- run hello-world

```bash
root@template-docker-in-lxc:/etc/apt# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 

```


### Install docker-compose

```shell

sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose


```

### Fixed root shell path  cause login failed

When I setup zsh and set wrong zsh.then I cannot login and ssh
The error like this. I can not boot to single user mode. 
```shell
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.18-12-pve x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Ubuntu's Kubernetes 1.14 distributions can bypass Docker and use containerd
   directly, see https://bit.ly/ubuntu-containerd or try it now with

     snap install microk8s --classic

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Cannot execute zsh: No such file or directory
```

**Solution:** I wounder that is a a command like docker command `docker exec -it $dockername /bin/sh`
Bingo! Here we go:

```shell
lxc-attach -n $lxc_pid /bin/sh
#for example:
lxc-attach -n 103 /bin/sh
vi /etc/password
#then change root shell path to /bin/sh

```
After reboot and I can login again.


## Reference

- [Install Docker Compose](https://docs.docker.com/compose/install/)
- [LXC 简介](https://jin-yang.github.io/post/linux-container-lxc-introduce.html)

