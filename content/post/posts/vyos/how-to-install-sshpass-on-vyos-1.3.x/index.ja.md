---
title: "How to Install Sshpass on Vyos 1.3.x"
date: 2022-11-14T09:09:33+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=81ff83c3
categories:
  - vyos
tags:
  - vyos 
  - sshpass
  - debian
  - networking
slug:
  - vyos
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

# How to Install SSHPASS on VyOS 1.3.x


how-to-install-sshpass-on-
vyos-1.3.x
Since vyos 1.3.x base on debian 10, so we can install sshpass 1.06 for debian on vyos 1.3.x

## Download and install sshpass 1.06 for debian 10
 
```bash
 wget http://ftp.de.debian.org/debian/pool/main/s/sshpass/sshpass_1.06-1_amd64.deb

sudo dpkg -i sshpass_1.06-1_amd64.deb
```


## Run sshpass

```bash

# run command via ssh without inactively
sshpass -p 'ssh_password' ssh -t root@ssh_host -p 22 'uptime'

# Copy files to remote 
sshpass -p 'ssh_password' scp -P 22 myfile.txt gcadmin@ssh_host

# loop running command
for i in 192.168.11.5{1..3};do sshpass -p 'ssh_password' ssh -t root@$i -p 22 'uptime';done

# Loop copy files
for i in host1 host2 host3;do echo $i && sshpass -p 'ssh_password' scp -P 22 myfile.txt gcadmin@$i;done
  

```

