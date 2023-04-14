---
title: "How to Install Ansible on Debian 11"
date: 2023-04-14T18:20:02+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=e642d07c
categories:
  - Blog
tags:
  - debian
  - ansible
  - pve
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

# Install ansible on debian11

## Install ansible with apt 
```bash
cat << EOF > /etc/apt/sources.list.d/ansible.list
deb http://ppa.launchpad.net/ansible/ansible/ubuntu focal main
EOF


apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
apt update 
apt install ansible -y
ansible --version
```


## Install ansible using pip

```bash
apt-get install python3 python3-pip -y
pip install ansible
```


## Reference

  - [how-to-install-and-use-ansible-on-debian-11](https://www.howtoforge.com/how-to-install-and-use-ansible-on-debian-11/)

