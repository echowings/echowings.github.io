---
title: "Bcache"
date: 2021-03-09T16:20:42+08:00
lastmode: 2021-03-09T16:20:42+08:00
draft: false
tags: [ "bcache", "pve","hci","debian" ]
categories: ["linux"]
reward: true
mathjax: true
---

# How to operate becache

Recently I just try to add bcache with zfs and ceph. So I just wirte down it.


## Install bcache-tools
Since bcache implement in Linux kernel. So it will default enabled. We just need to install `bcache-tools` to operate bcache.

  ```bash
apt install -y bcache-tools
  ```
## Configure bcache

```bash
# make a cache disk for bcache .It must be a ssd disk.
make-bcache -C /dev/nvme1n1 

# make a disk as backend disk.It must be a slow sata/sas disk.
make-bcache -B /dev/sda
#if you have a lot of disks like me.your can add backend disk as this.
make-bcache -B /dev/sd{a..f}
```


###Check bcache status
```bash
lsblk
```

### Add bcache manually
```bash
#show `cset.uuid
root@xa-autotest-hci01:~# bcache-super-show /dev/sda
sb.magic		ok
sb.first_sector		8 [match]
sb.csum			ED8FD98E20A2E91E [match]
sb.version		1 [backing device]

dev.label		(empty)
dev.uuid		e7b350fe-3496-47e2-a276-5faa5637f8a5
dev.sectors_per_block	1
dev.sectors_per_bucket	1024
dev.data.first_sector	16
dev.data.cache_mode	1 [writeback]
dev.data.cache_state	1 [clean]

cset.uuid		d3fcd781-a10a-453a-96dd-6eba13abfc5b
```
####Add cache disk

```bash
echo "d3fcd781-a10a-453a-96dd-6eba13abfc5b" > /sys/block/bcache0/bcache/attach
```

if you have a lot of disk, you can add it with for loop.

```bash
for i in range{0..5}; do `echo "d3fcd781-a10a-453a-96dd-6eba13abfc5b" > /sys/block/bcache$(expr $i)/bcache/attach`; done
```

#### Delete cache disk
 if you want to delete bcache disk

```bash
echo "d3fcd781-a10a-453a-96dd-6eba13abfc5b" > /sys/block/bcache0/bcache/detach
# delete a lot disk 
for i in range{0..5}; do `echo "d3fcd781-a10a-453a-96dd-6eba13abfc5b" > /sys/block/bcache$(expr $i)/bcache/detach`; done
```
#### unrigister cache disk

```bash
umount /dev/bcach0
echo 1 > /sys/block/bcache0/bcache/stop
```


 If you onle have one nvme disk and you can add them all in one
make-bcache -C /dev/nvme1n1 -B /dev/sd{a..f} --writeback

```

## Change bcache write policy

```bash
make-bcache -B/dev/
```

## Manually resigter udev disk

```bash
echo /dev/sdb > /sys/fs/bcache/register
```

## Change bcache write mode
When you run command like `make-bcache -C /dev/nvme1n1 -B /dev/sda` it will be set `write through` mode. If you have UPS and you can change it to `writeback` mode.

```bash
echo writeback > /sys/block/bcache0/bcache/cache_mode
```

## Check bcache status

```bash
cat /sys/block/bcache0/bcache/state
no cache: this means you have not attached a caching device to your backing bcache device;
clean: this means everything is ok. The cache is clean;
dirty: this means everything is setup fine and that you have enabled writeback and that the cache is dirty;
inconsistent: you are in trouble because the backing device is not in sync with the caching device;

bcache-status
lsblk
```


## bcache optimized

```bash
#modify cache policy 
cat /sys/block/bcache0/bcache/cache_mode
echo writeback > /sys/block/bcache0/bcache/cache_mode
#Disable ip track
echo 0 > /sys/fs/bcache/$CacheSetUUID/congested_read_threshold_us
echo 0 > /sys/fs/bcache/$CacheSetUUID/congested_write_threshold_us
```

## disable bcache

```bash
#check `CACHE_SET_UUID`
ls /sys/fs/bcache
#Unregister ssd cache disk
echo 1 >/sys/fs/bcache/06e33354-7c21-4c52-990d-1a653617ab20/unregister

#disable backend disk
echo 1 > /sys/block/sdb/bcache/stop
#write back cache to backend disk
echo 0 > /sys/block/bcache0/bcache/writeback_percent
```


##Reference
  - [A block layer cache (bcache)](https://www.geek-share.com/detail/2719485821.html)
  - [bcache狀態和配置檔案詳細介紹(翻譯自官網）](https://www.itread01.com/content/1548985342.html)
  - [bcache手册](https://webcache.googleusercontent.com/search?q=cache:9eiyDtuwt9AJ:https://wiki2.xbits.net:4430/storage:%25E7%25BC%2593%25E5%25AD%2598%25E6%258A%2580%25E6%259C%25AF:bcache:bcache%25E6%2589%258B%25E5%2586%258C+&cd=1&hl=en&ct=clnk&gl=hk)
  - [Linux下SSD缓存加速之bcache试用](https://www.mr-mao.cn/archives/linux-bcache-performance-testing.html)
  - [bcache / 如何使用bcache构建LVM,软RAID / 如何优化bcache](https://billtian.github.io/digoal.blog/2016/09/19/01.html)