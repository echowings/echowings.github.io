---
title: "Proxmox Ve Network Adapter Pass Through"
date: 2018-12-07T00:11:11+08:00
lastmod: 2018-12-24T16:01:23+08:00
draft: false
tags: ["PVE","pass through"]
categories: ["virtualization"]
author: "Steve Dong"
reward: true
---

# Proxmox Network Adapter Pass Through

Long time ago, I konw virtualization is go very good solution to server. But how about router build in virtualization? After tried to do it, I was failed. because virtual switch made networking very slow. So i just use dedicate server as a router.

Recently days, I just learn proxmox and find pci-e pass through function. I think maybe it is way to make routing virtualization.

Here are what I do:

## Enable VT-d and X2APic Mode
Configure BIOS to enable VT-d and X2APic

## Chang Grub boot settings

### Inel CPU
* Edit grub configure.
* Inter CPU
* Edit grub

* Edit `/etc/default/grub`, then change
`GRUB_CMDLINE_LINUX_DEFAULT="quiet"` to 
`GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"`

* Update grub

`update-grub`

### AMD CPU

* Edit grub file. 
* Edit `/etc/default/grub`

* Change `GRUB_CMDLINE_LINUX_DEFAULT="quiet" `to `GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"`

* Update grub

  `update-grub`

##Change Required modules

* Edit /etc/modules

Add

```bash
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

* Check IOMMU enable or not

```bash
dmesg | grep ecap
```

* script to make sure IOMMU is enabled

```bash
#!/bin/sh
if [ $(dmesg | grep ecap | wc -l) -eq 0 ]; then
    echo "No interrupt remapping support found"
    exit 1
fi

for i in $(dmesg | grep ecap | awk '{print $NF}'); do
    if [ $(( (0x$i & 0xf) >> 3 )) -ne 1 ]; then
        echo "Interrupt remapping not supported"
        exit 1
fi
done
```

* Verify IOMMU isolation

```shell
find /sys/kernel/iommu_groups/ -type l
```

## Add PCI-E Device to VM

* Run `lspci` to check address of network pci-e adapter device. like this 

``` bash
lspci
```

```
hostpci0: 01:00.0
```
* Add configure of yoru VM `/etc/pve/qemu-server/<vmid>.conf`
* Add configure like this to set pci-expres passthrough.

```bash  
  machine: q35
  hostpci0: 00:00.0,pcie=1
```

## Reference

* [Pci passthrough](https://pve.proxmox.com/wiki/Pci_passthrough) 
