---
title: "How to Enable Proxmoxve Nested Virtualization"
date: 2019-02-14T11:44:28+08:00
draft: false
tags: ["pve"]
categories: ["virtualization"]
reward: true
---
# How to enable Proxmox VE nested virtualization

To enable nested virtualization on Proxmox VE, we need to stop all the vm and enable nested virtualization to linux kernel. then change vm settings

## Stop all the vm on proxmox VE

  - stop all virtual machines with shell command. 

  ``` bash
root@xa-sys-pve01:~# pvesh create /nodes/localhost/stopall
Stopping VM 116 (timeout = 180 seconds)
Stopping VM 115 (timeout = 180 seconds)
Stopping CT 112 (timeout = 60 seconds)
Stopping CT 111 (timeout = 60 seconds)
Stopping CT 108 (timeout = 60 seconds)
Stopping CT 106 (timeout = 60 seconds)
Stopping VM 101 (timeout = 180 seconds)
UPID:xa-sys-pve01:0036EA66:01992199:5CB98D6C:stopall::root@pam:
  ```
  - stop all the virtual machine with admin web.
 
 ```
 bulk stop
 ```

## Enable nested module in linux kernel

  1. Check nested support status

  ``` bash
  cat /sys/module/kvm_intel/parameters/nested
  N
  ```
  2. enable nested support on.

  ``` bash
  #for intel cpu
  echo "options kvm-intel nested=Y" > /etc/modprobe.d/kvm-intel.conf
  
  #for amd cpu
  echo "options kvm-amd nested=1" > /etc/modprobe.d/kvm-amd.conf
  ```
  
  3. Reload the kernel module.

  ``` bash
  modprobe -r kvm_intel
  modprobe kvm_intel
  ```
  4. Check again.

  ``` bash
  cat /sys/module/kvm_intel/parameters/nested                    
  Y
  ```
  
  ## Change VM cpu type
  ~~~Change VM cpu type to host and boot it up.~~~
  
  ```bash
  vi /etc/pve/qemu-server/xxx.conf
  ```
  Add a args to the config file
  
  ``` vim
  #intel cpu
  args: -cpu host,+vmx
  
  #amd cpu
  args: -cpu host,+svm
  ```


## Reference
  * [Nested Virtualization](https://pve.proxmox.com/wiki/Nested_Virtualization)
