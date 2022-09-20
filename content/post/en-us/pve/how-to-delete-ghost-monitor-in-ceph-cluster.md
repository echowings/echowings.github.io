---
title: "How to Delete Ghost Monitor in Ceph Cluster"
date: 2021-07-01T14:06:58+08:00
lastmode: 2021-07-01T14:06:58+08:00
draft: false
tags: [ "hci", "pve","ceph" ]
categories: ["pve"]
reward: true
mathjax: true
---

# How to delete ghost monitor in ceph cluster
---

Today , after I let a new server join pve cluster. ceph mon is stopped. I want to start or delete it , I always got a failed notification. After search and try to fix it. At last I fixed it.Here is what I had do

## Issue description

```bash
root@xa-autotest-hci04:~# pveceph createmon
monitor 'xa-autotest-hci04' already exists
root@xa-autotest-hci04:~# pveceph mon destroy xa-autotest-hci04
no such monitor id 'xa-autotest-hci04'
```

## How to fixed it

```bash
#stop ceph-mon@xa-autotest-hci04 service
systemctl stop ceph-mon@xa-autotest-hci04

#Disable ceph-mon@xa-autotest-hci04 service
systemctl disable ceph-mon@xa-autotest-hci04


pveceph purge
rm -rf  /var/lib/ceph/mon/ceph-xa-autotest-hci04

pveceph createmon
#Created symlink /etc/systemd/system/ceph-mon.target.wants/ceph-mon@xa-autotest-hci04.service -> /lib/systemd/system/ceph-mon@.service.

```

