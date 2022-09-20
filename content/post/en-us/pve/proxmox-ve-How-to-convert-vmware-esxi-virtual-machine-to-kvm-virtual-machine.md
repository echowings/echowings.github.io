---
title: "Proxmox VE - How to Convert Vmware Esxi VM to proxmox-VE VM"
date: 2019-04-26T22:12:14+08:00
draft: false
tags: ["pve"]
categories: ["virtualization"]
auther: "steve dong"
reward: true
---


# How to convert vmware esix VM to proxmox-VE VM

## V2V
Convert vmware virtual machines to kvm virtual machine is easy.

### Convert windows VM 
  - Remove VWware tools
  Uninstall vmware tools and reboot vm.
  
  - Switch disk mode to IDE

  - Download [Mergeide](https://pve.proxmox.com/wiki/File:Mergeide.zip), unzip it and run it,then reboot your vm. or you can save follow code as `mergeide.reg`.


``` shell
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\primary_ide_channel]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="atapi"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\secondary_ide_channel]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="atapi"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\*pnp0600]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="atapi"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\*azt0502]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="atapi"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\gendisk]
"ClassGUID"="{4D36E967-E325-11CE-BFC1-08002BE10318}"
"Service"="disk"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#cc_0101]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_0e11&dev_ae33]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1039&dev_0601]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1039&dev_5513]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1042&dev_1000]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_105a&dev_4d33]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1095&dev_0640]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1095&dev_0646]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1095&dev_0646&REV_05]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1095&dev_0646&REV_07]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1095&dev_0648]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1095&dev_0649]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1097&dev_0038]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_10ad&dev_0001]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_10ad&dev_0150]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_10b9&dev_5215]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_10b9&dev_5219]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_10b9&dev_5229]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="pciide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_1106&dev_0571]
"Service"="pciide"
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_1222]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_1230]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_2411]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_2421]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_7010]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_7111]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\CriticalDeviceDatabase\pci#ven_8086&dev_7199]
"ClassGUID"="{4D36E96A-E325-11CE-BFC1-08002BE10318}"
"Service"="intelide"

;Add driver for Atapi (requires Atapi.sys in Drivers directory)

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\atapi]
"ErrorControl"=dword:00000001
"Group"="SCSI miniport"
"Start"=dword:00000000
"Tag"=dword:00000019
"Type"=dword:00000001
"DisplayName"="Standard IDE/ESDI Hard Disk Controller"
"ImagePath"=hex(2):53,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,44,00,\ 
  52,00,49,00,56,00,45,00,52,00,53,00,5c,00,61,00,74,00,61,00,70,00,69,00,2e,\ 
  00,73,00,79,00,73,00,00,00

;Add driver for intelide (requires intelide.sys in drivers directory)

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IntelIde]
"ErrorControl"=dword:00000001
"Group"="System Bus Extender"
"Start"=dword:00000000
"Tag"=dword:00000004
"Type"=dword:00000001
"ImagePath"=hex(2):53,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,44,00,\ 
  52,00,49,00,56,00,45,00,52,00,53,00,5c,00,69,00,6e,00,74,00,65,00,6c,00,69,\ 
  00,64,00,65,00,2e,00,73,00,79,00,73,00,00,00


;Add driver for Pciide (requires Pciide.sys and Pciidex.sys in Drivers directory)

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PCIIde]
"ErrorControl"=dword:00000001
"Group"="System Bus Extender"
"Start"=dword:00000000
"Tag"=dword:00000003
"Type"=dword:00000001
"ImagePath"=hex(2):53,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,44,00,\ 
  52,00,49,00,56,00,45,00,52,00,53,00,5c,00,70,00,63,00,69,00,69,00,64,00,65,\ 
  00,2e,00,73,00,79,00,73,00,00,00

  
```



  -  Copy *-flat.vmdk file to proxmxo ve host.

  Shutdown your vm.

  ``` shell
  
  scp root@xx.xx.xx.xx:/vmfs/volumes/datastore/source-vm/source-vm-flat.vmdk /var/lib/vz/images/soure-vm-flate.vmdk


 ```

  -  Convert vmdk file to qcow2

  ```shell
  		  	
  qemu-img convert -f raw source-vm-flate.vmdk -O qcow2 source-vm-flate.qcow2
     	
  
  ```
 
 
  - Create a  vm  which os type is same as original vm and replace the disk with source-vm-flate.qcow2

  - Download [virtio-win.iso](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso)  and upload iso image to proxmox ve host storage.
  - Mount iso in cdrom device in vm settings.
  - Add and virtoIO Block disk and install driver from cdrom. Then reboot the vm.
  - Change `Bus/Device` in `Hard Disk` to `VirtIO Block`.
  - change netork device  model type to `VirtIO(paravirtualized)`.
  - delete the added the last disk.


  


### Convert Linux or freebsd VM

  -  Copy *-flat.vmdk file to proxmxo ve host.

  Shutdown your vm.

 
  ``` shell
scp root@xx.xx.xx.xx:/vmfs/volumes/datastore/source-vm/source-vm-flat.vmdk /var/lib/vz/images/soure-vm-flate.vmdk
	
```

  -  Convert vmdk file to qcow2

  ```shell
  	
  qemu-img convert -f raw source-vm-flate.vmdk -O qcow2 source-vm-flate.qcow2
  	
  ```
 
 
  - Create a  vm  which os type is same as original vm and replace the disk with source-vm-flate.qcow2
  - Change `Bus/Device` in `Hard Disk` to `VirtIO Block`.
  - change netork device  model type to `VirtIO(paravirtualized)`.

  



## Reference

  - [Migration of servers to Proxmox VE](https://pve.proxmox.com/wiki/Migration_of_servers_to_Proxmox_VE)
  - [Windows VirtIO Drivers](https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers)