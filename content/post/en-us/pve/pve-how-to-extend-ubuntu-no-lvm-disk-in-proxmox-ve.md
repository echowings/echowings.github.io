---
title: "How to Extend Ubuntu No Lvm Disk in Proxmox VE"
date: 2019-05-28T15:20:53+08:00
draft: false
tags: ["pve", "storage"]
categories: ["virtualization"]
reward: true

---

# How to resize  disk(no lvm) in proxmox ve


Today I met a problem that I need to extend disk image for a ubuntu server which is a virtual machine hosted in proxmox ve.

## Resizing disk images

First thing first, I need resize the disk images of the virtual machine.I can do it both in command line or via web management interface.

- Extend disk size of virtual machine via shell command

```shell
# Extend disk size of virtual machine with vid 136 with 100G
qm resize 136 virtio0 +100G

```
 - Extend disk size of virtual machien via web interface

 

`"Datacenter" | "$Node" | $VIRTUAL_MACHINE | "HARDWARE" | "HARD DISK(SCSI0)" | "RESIZE DISK" | "size Increment(GiB) | "100"  | "Resize disk"`


## Enlarge the partition in the virtual disk

Login the virtual machine and run command as show below:

-  Show disk partition

``` shell
# show disk device
ls -al /dev/sda*
```

- Run parted with command `parted /dev/sda`

``` shell
root@xxx-xxxx-server:~# parted /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
```

-  print partitions. `print`

```shell
(parted) print
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 389GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  2097kB  1049kB                     bios_grub
 2      30720kB  30GB   30GB   ext4
```

-  Resize partition with command `resizepart`.

```shell
(parted) resizepart
```

-  select partition which I wanted to resize. `2`

```shell
Partition number? 2
```

-  Enter `y` to continue.

```shell
Warning: Partition /dev/sda2 is being used. Are you sure you want to continue?
Yes/No? y
```

-  Enter size `200GB`

```shell
End?  [30GB]? 200GB
(parted) print
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sda: 389GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  2097kB  1049kB                     bios_grub
 2      2097kB  200GB   200GB   ext4
```

 - Quit parted with `quit`.

```shell
(parted) :quit
```


## Enlarge partition with `resize2f`


```shell
resize2f /dev/sda2 99G
# Show size of partition
df -h 

```





## Swell

  - [Shrink Qcow2 Disk Files](https://pve.proxmox.com/wiki/Shrink_Qcow2_Disk_Files)
  - [Resize disks](https://pve.proxmox.com/wiki/Resize_disks)
  - [How to Extend primary partition(/dev/sda1) in linux?
](https://superuser.com/questions/441379/how-to-extend-primary-partition-dev-sda1-in-linux)