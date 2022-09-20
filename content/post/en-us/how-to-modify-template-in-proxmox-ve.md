---
title: "How to Modify Template in Proxmox Ve"
date: 2019-07-18T12:23:01+08:00
lastmode: 2019-07-18T12:23:01+08:00
draft: false
tags: [ "pve"]
categories: ["virtualization"]
reward: true
mathjax: true
---

# How to Modify Template im Proxox VE


Sometime I wonder how to convert template back to vm and modify it and convert it back to vm? Here is the solution


## Convert Template to VM

Delete `template: 1` in vm config.

```bash
sed -i 's/template:\ 1//g' /etc/pve/qemu-server/{vm-id}.conf

```

## Change access permission of VM disk 

```bash
cd /var/lib/vz/images/1{VM-ID}/
chattr -i *
chmod 755 *

```

## Then covert it back to template


