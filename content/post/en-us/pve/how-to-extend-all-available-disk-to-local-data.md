---
title: "How to Extend All Available Disk to Local Data"
date: 2020-09-04T09:11:30+08:00
lastmode: 2020-09-04T09:11:30+08:00
draft: false
tags: [ "pve", "lvm" ]
categories: ["virtualization"]
reward: true
mathjax: true
---

# How to extend all available disk to local-data

After install Proxmove VE, the local-data only have about 50GB disk availabe. But how to let all availabld disk space to the local-data? Here we go:

```bash
# Remove local-data
lvremove /dev/pve/data

# Extend all available disk to /dev/pve/root
lvresize -l +100%FREE /dev/pve/root

#recreate pve-root
resize2fs /dev/mapper/pve-root
```


## Reference

  - [x] [Proxmox – Remove LVM local-data](https://artofawareness.wordpress.com/2017/03/06/proxmox-remove-lvm-local-data/)  



