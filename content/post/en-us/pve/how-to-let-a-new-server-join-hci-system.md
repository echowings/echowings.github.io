---
title: "How to Let a New Server Join Hci System"
date: 2021-07-01T06:38:10+08:00
lastmode: 2021-07-01T06:38:10+08:00
draft: false
tags: [ "pve", "hci","ceph", "bcache" ]
categories: ["pve"]
reward: true
mathjax: true
---

##How to let a new server join the exist HCI cluster

### 1. Insall the pve on the server
--skip---

### 2. Let the new server join the pve cluster

```bash
#ssh login the server
pvecm add 10.32.4.37
```
### 3. configure NTP server

```bash
cat >> /etc/systemd/timesyncd.conf << EOF
NTP=192.168.0.2 192.168.0.3
EOF
# Restart systemd-timesyncd service to apply settings
systemctl restart systemd-timesyncd
```
### 4. Install ceph

```bash
pveceph install
```

### 5. Partition nvme disk
  - partitin 1-6 for block.wal(ceph)
  - partition 7-12 for block.db(ceph)
  - partition 13-18 for bcache 


```bash
parted -s /dev/nvme1n1 mktable gpt
parted -s /dev/nvme1n1 mkpart primary 1M 10G
parted -s /dev/nvme1n1 mkpart primary 10G 20G
parted -s /dev/nvme1n1 mkpart primary 20G 30G
parted -s /dev/nvme1n1 mkpart primary 30G 40G
parted -s /dev/nvme1n1 mkpart primary 40G 50G
parted -s /dev/nvme1n1 mkpart primary 50G 60G

parted -s /dev/nvme1n1 mkpart primary 60G 110G
parted -s /dev/nvme1n1 mkpart primary 110G 160G
parted -s /dev/nvme1n1 mkpart primary 160G 210G
parted -s /dev/nvme1n1 mkpart primary 210G 260G
parted -s /dev/nvme1n1 mkpart primary 260G 310G
parted -s /dev/nvme1n1 mkpart primary 310G 360G

parted -s /dev/nvme1n1 mkpart primary 360G 433G
parted -s /dev/nvme1n1 mkpart primary 433G 506G
parted -s /dev/nvme1n1 mkpart primary 506G 579G
parted -s /dev/nvme1n1 mkpart primary 579G 652G
parted -s /dev/nvme1n1 mkpart primary 652G 725G
parted -s /dev/nvme1n1 mkpart primary 725G 800G

parted -s /dev/nvme2n1 mktable gpt
parted -s /dev/nvme2n1 mkpart primary 1M 10G
parted -s /dev/nvme2n1 mkpart primary 10G 20G
parted -s /dev/nvme2n1 mkpart primary 20G 30G
parted -s /dev/nvme2n1 mkpart primary 30G 40G
parted -s /dev/nvme2n1 mkpart primary 40G 50G
parted -s /dev/nvme2n1 mkpart primary 50G 60G

parted -s /dev/nvme2n1 mkpart primary 60G 110G
parted -s /dev/nvme2n1 mkpart primary 110G 160G
parted -s /dev/nvme2n1 mkpart primary 160G 210G
parted -s /dev/nvme2n1 mkpart primary 210G 260G
parted -s /dev/nvme2n1 mkpart primary 260G 310G
parted -s /dev/nvme2n1 mkpart primary 310G 360G

parted -s /dev/nvme2n1 mkpart primary 360G 433G
parted -s /dev/nvme2n1 mkpart primary 433G 506G
parted -s /dev/nvme2n1 mkpart primary 506G 579G
parted -s /dev/nvme2n1 mkpart primary 579G 652G
parted -s /dev/nvme2n1 mkpart primary 652G 725G
parted -s /dev/nvme2n1 mkpart primary 725G 800G
```
### 6. list disk by-id

```bash
# list sas disk by id
ls -al | grep wwn
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000c500943614ab -> ../../sdl
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca0801eb5c8 -> ../../sdc
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca080248ff8 -> ../../sdh
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca08024aca8 -> ../../sdk
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca08024f0d0 -> ../../sdi
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca080253734 -> ../../sdj
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca0802601cc -> ../../sdf
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca0802735ec -> ../../sda
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca0802747c8 -> ../../sdd
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca0802afad0 -> ../../sdb
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca0802ba6ac -> ../../sdg
lrwxrwxrwx 1 root root    9 Jul  1 05:49 wwn-0x5000cca080326bd0 -> ../../sde
```

```bash
#list nvme disk by uuid
ls -al | grep nvme-MT
lrwxrwxrwx 1 root root   13 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN -> ../../nvme1n1
lrwxrwxrwx 1 root root   15 Jul  1 06:07 nvme-MT0800KEXUU_CVFT73520058800MGN-part1 -> ../../nvme1n1p1
lrwxrwxrwx 1 root root   16 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part10 -> ../../nvme1n1p10
lrwxrwxrwx 1 root root   16 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part11 -> ../../nvme1n1p11
lrwxrwxrwx 1 root root   16 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part12 -> ../../nvme1n1p12
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN-part13 -> ../../nvme1n1p13
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN-part14 -> ../../nvme1n1p14
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN-part15 -> ../../nvme1n1p15
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN-part16 -> ../../nvme1n1p16
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN-part17 -> ../../nvme1n1p17
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT73520058800MGN-part18 -> ../../nvme1n1p18
lrwxrwxrwx 1 root root   15 Jul  1 06:07 nvme-MT0800KEXUU_CVFT73520058800MGN-part2 -> ../../nvme1n1p2
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part3 -> ../../nvme1n1p3
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part4 -> ../../nvme1n1p4
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part5 -> ../../nvme1n1p5
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part6 -> ../../nvme1n1p6
lrwxrwxrwx 1 root root   15 Jul  1 06:07 nvme-MT0800KEXUU_CVFT73520058800MGN-part7 -> ../../nvme1n1p7
lrwxrwxrwx 1 root root   15 Jul  1 06:07 nvme-MT0800KEXUU_CVFT73520058800MGN-part8 -> ../../nvme1n1p8
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT73520058800MGN-part9 -> ../../nvme1n1p9
lrwxrwxrwx 1 root root   13 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN -> ../../nvme2n1
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT7352005S800MGN-part1 -> ../../nvme2n1p1
lrwxrwxrwx 1 root root   16 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part10 -> ../../nvme2n1p10
lrwxrwxrwx 1 root root   16 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part11 -> ../../nvme2n1p11
lrwxrwxrwx 1 root root   16 Jul  1 06:10 nvme-MT0800KEXUU_CVFT7352005S800MGN-part12 -> ../../nvme2n1p12
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN-part13 -> ../../nvme2n1p13
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN-part14 -> ../../nvme2n1p14
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN-part15 -> ../../nvme2n1p15
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN-part16 -> ../../nvme2n1p16
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN-part17 -> ../../nvme2n1p17
lrwxrwxrwx 1 root root   16 Jul  1 05:49 nvme-MT0800KEXUU_CVFT7352005S800MGN-part18 -> ../../nvme2n1p18
lrwxrwxrwx 1 root root   15 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part2 -> ../../nvme2n1p2
lrwxrwxrwx 1 root root   15 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part3 -> ../../nvme2n1p3
lrwxrwxrwx 1 root root   15 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part4 -> ../../nvme2n1p4
lrwxrwxrwx 1 root root   15 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part5 -> ../../nvme2n1p5
lrwxrwxrwx 1 root root   15 Jul  1 06:10 nvme-MT0800KEXUU_CVFT7352005S800MGN-part6 -> ../../nvme2n1p6
lrwxrwxrwx 1 root root   15 Jul  1 06:08 nvme-MT0800KEXUU_CVFT7352005S800MGN-part7 -> ../../nvme2n1p7
lrwxrwxrwx 1 root root   15 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part8 -> ../../nvme2n1p8
lrwxrwxrwx 1 root root   15 Jul  1 06:09 nvme-MT0800KEXUU_CVFT7352005S800MGN-part9 -> ../../nvme2n1p9
```
after sort the data ,will will got this list

```bash
#sas disk
/dev/disk/by-id/wwn-0x5000cca0802735ec
/dev/disk/by-id/wwn-0x5000cca0802afad0
/dev/disk/by-id/wwn-0x5000cca0801eb5c8
/dev/disk/by-id/wwn-0x5000cca0802747c8
/dev/disk/by-id/wwn-0x5000cca080326bd0
/dev/disk/by-id/wwn-0x5000cca0802601cc
/dev/disk/by-id/wwn-0x5000cca0802ba6ac
/dev/disk/by-id/wwn-0x5000cca080248ff8
/dev/disk/by-id/wwn-0x5000cca08024f0d0
/dev/disk/by-id/wwn-0x5000cca080253734
/dev/disk/by-id/wwn-0x5000cca08024aca8
/dev/disk/by-id/wwn-0x5000c500943614ab

# nvme
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part1 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part2 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part3 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part4 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part5 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part6 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part7 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part8 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part9 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part10
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part11
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part12
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part1 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part2 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part3 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part4 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part5 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part6 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part7 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part8 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part9 
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part10
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part11
/dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part12
```

### 7. Make Bcache disk

```bash
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part13 -B /dev/disk/by-id/wwn-0x5000cca0802735ec --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part14 -B /dev/disk/by-id/wwn-0x5000cca0802afad0 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part15 -B /dev/disk/by-id/wwn-0x5000cca0801eb5c8 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part16 -B /dev/disk/by-id/wwn-0x5000cca0802747c8 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part17 -B /dev/disk/by-id/wwn-0x5000cca080326bd0 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT73520058800MGN-part18 -B /dev/disk/by-id/wwn-0x5000cca0802601cc --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part13 -B /dev/disk/by-id/wwn-0x5000cca0802ba6ac --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part14 -B /dev/disk/by-id/wwn-0x5000cca080248ff8 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part15 -B /dev/disk/by-id/wwn-0x5000cca08024f0d0 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part16 -B /dev/disk/by-id/wwn-0x5000cca080253734 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part17 -B /dev/disk/by-id/wwn-0x5000cca08024aca8 --writeback
make-bcache -C /dev/disk/by-id/nvme-MT0800KEXUU_CVFT7352005S800MGN-part18 -B /dev/disk/by-id/wwn-0x5000c500943614ab --writeback
```
### 8. Reboot to make bcache works
```bash
reboot now
```

### 9. create OSD 

```bash
ceph-volume lvm create --bluestore --data /dev/bcache0 --block.wal /dev/nvme1n1p1  --block.db /dev/nvme1n1p7
ceph-volume lvm create --bluestore --data /dev/bcache1 --block.wal /dev/nvme1n1p2  --block.db /dev/nvme1n1p8
ceph-volume lvm create --bluestore --data /dev/bcache2 --block.wal /dev/nvme1n1p3  --block.db /dev/nvme1n1p9
ceph-volume lvm create --bluestore --data /dev/bcache3 --block.wal /dev/nvme1n1p4  --block.db /dev/nvme1n1p10
ceph-volume lvm create --bluestore --data /dev/bcache4 --block.wal /dev/nvme1n1p5  --block.db /dev/nvme1n1p11
ceph-volume lvm create --bluestore --data /dev/bcache5 --block.wal /dev/nvme1n1p6  --block.db /dev/nvme1n1p12
ceph-volume lvm create --bluestore --data /dev/bcache6 --block.wal /dev/nvme2n1p1  --block.db /dev/nvme2n1p7
ceph-volume lvm create --bluestore --data /dev/bcache7 --block.wal /dev/nvme2n1p2  --block.db /dev/nvme2n1p8
ceph-volume lvm create --bluestore --data /dev/bcache8 --block.wal /dev/nvme2n1p3  --block.db /dev/nvme2n1p9
ceph-volume lvm create --bluestore --data /dev/bcache9 --block.wal /dev/nvme2n1p4  --block.db /dev/nvme2n1p10
ceph-volume lvm create --bluestore --data /dev/bcache10 --block.wal /dev/nvme2n1p5  --block.db /dev/nvme2n1p11
ceph-volume lvm create --bluestore --data /dev/bcache11 --block.wal /dev/nvme2n1p6  --block.db /dev/nvme2n1p12
```