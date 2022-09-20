---
title: "How to Import Disk to Ceph"
date: 2019-05-07T12:06:58+08:00
draft: false
tags: ["pve","ceph"]
categories: ["virtualization"]
auther: "steve dong"
reward: true
---



## Convert Disk to RAW

``` shell
#qcow2
qemu-img convert  -f qcow2  xxx.qcow2 -O raw xxxx.raw
#vmware
qemu-img convert -f vmdk xxx-flat.vmdk -O raw xxx.raw

```

## Import RAW to ceph



```shell
rbd list --pool ${ceph-pool-name}
rbd import ./xxx.raw --pool ${ceph-pool-name}
rbd rm ${old-file-rm} --pool ${ceph-pool-name]
rbd mv xxx.raw ${old-file-rm)

rbd list --poll ${ceph-pool-name}

```

## Reference

  - [将qcow2格式的虚拟机文件导入到ceph池里](https://blog.csdn.net/Packet886/article/details/80522425)




