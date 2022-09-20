---
title: "How to Extend Lvm Allocted PE With Freee PE"
date: 2019-01-08T11:47:31+08:00
draft: false
tags: ["lvm", "linux"]
categories: ["linux"]
author: "Steve Dong"
reward: true
---


## LVM EXTEND
How to extend lvm size use free PE

``` bash
# show volume group information
vgdisplay 
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <99.00 GiB
  PE Size               4.00 MiB
  Total PE              25343
  Alloc PE / Size       1024 / 4.00 GiB
  Free  PE / Size       24319 / <95.00 GiB
  VG UUID               KblInT-Z6k6-1NM3-ybyV-eufy-y1vc-BiIoEM
```
Remember ```24319``` from ```Free  PE / Size       24319 / <95.00 GiB``` 

## list logical volume 
``` bash
lvdisplay
```
``` bash
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                Pi0Id7-riMg-BtmN-sfdF-gIjX-KrCp-TxxCBt
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2018-12-12 03:43:02 +0000
  LV Status              available
  # open                 1
  LV Size                4.00 GiB
  Current LE             1024
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
Remember LV Path: ```/dev/ubuntu-vg/ubuntu-lv```

## Extend Free PE to lvm

``` bash
lvextend -l +24319 /dev/ubuntu-vg/ubuntu-lv
```

``` bash
root@template-ubutu1804:/home/autotest# vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <99.00 GiB
  PE Size               4.00 MiB
  Total PE              25343
  Alloc PE / Size       25343 / <99.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               KblInT-Z6k6-1NM3-ybyV-eufy-y1vc-BiIoEM
  
resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.44.1 (24-Mar-2018)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 13
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 25951232 (4k) blocks long. 
```

## Check volume group information

``` bash
vgdisplay
```

``` bash
root@template-ubutu1804:/home/autotest# vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <99.00 GiB
  PE Size               4.00 MiB
  Total PE              25343
  Alloc PE / Size       25343 / <99.00 GiB
  Free  PE / Size       0 / 0
```

You will see ```Alloc PE``` size is ```25343```, size of `Free PE` is `0`.

## Brief of command

``` bash
vgdisplay 
lvdisplay
lvextend -l +24319 /dev/ubuntu-vg/ubuntu-lv
resize2fs /dev/ubuntu-vg/ubuntu-lv
vgdisplay
lvdisplay
```


