---
title: "Proxmox Ve How to Fixed Upgrade Error"
date: 2019-05-15T12:11:14+08:00
draft: false
tags: ["pve"]
categories: ["virtualization"]
reward: true
---


# How solove Proxmox VE 5.4 upgrade issue

Today I'm trying to upgrade proxmox ve 5.3. to proxmox ve 5.4 version. But I failed. After install and uninstall proxmox-manager, Force to reboot and the node has kernel panic error. Then force reboot the node. At grub menu select second line to select an old pve kernel to boot proxmox-ve.



## ERROR 1:"No space left on device" error on latest upgrade

If  you met issue like this:

``` bash

#gzip: stdout: No space left on device
#E: mkinitramfs failure find 141 cpio 141 gzip 1
#update-initramfs: failed for /boot/initrd.img-4.15.18-10-pve with 1.
#run-parts: /etc/kernel/postinst.d/initramfs-tools exited with return code 1
#Failed to process /etc/kernel/postinst.d at /var/lib/dpkg/info/pve-#kernel-4.15.18-10-pve.postinst line 19.
#dpkg: error processing package pve-kernel-4.15.18-10-pve (--configure):
# subprocess installed post-installation script returned error exit status 2
#Setting up pve-xtermjs (3.10.1-1) ...
#Setting up libpve-storage-perl (5.0-36) ...
#dpkg: dependency problems prevent configuration of pve-kernel-4.15:
# pve-kernel-4.15 depends on pve-kernel-4.15.18-10-pve; however:
#  Package pve-kernel-4.15.18-10-pve is not configured yet.

#dpkg: error processing package pve-kernel-4.15 (--configure):
# dependency problems - leaving unconfigured
#Setting up libpve-guest-common-perl (2.0-19) ...
#Setting up qemu-server (5.0-45) ...
#Setting up pve-ha-manager (2.0-6) ...
#Setting up pve-container (2.0-33) ...
#Setting up pve-manager (5.3-8) ...
#Processing triggers for initramfs-tools (0.130) ...
#update-initramfs: Generating /boot/initrd.img-4.15.18-9-pve

#gzip: stdout: No space left on device
#E: mkinitramfs failure find 141 cpio 141 gzip 1
#update-initramfs: failed for /boot/initrd.img-4.15.18-9-pve with 1.
#dpkg: error processing package initramfs-tools (--configure):
# subprocess installed post-installation script returned error exit status 1
#Processing triggers for pve-ha-manager (2.0-6) ...
#Processing triggers for systemd (232-25+deb9u8) ...
#Errors were encountered while processing:
# pve-kernel-4.15.18-10-pve
# pve-kernel-4.15
# initramfs-tools
#E: Sub-process /usr/bin/dpkg returned an error code (1)

```
You can fixed it with command like this.
### Solution:
 
List all of pve-kernel, Then remove pve-kernel except pve-kernel current your system running.
 
 ```bash
 # Check status of mounted filesytem 
 df -h
 # Check lvm status
 lvs -a
 # List installed pve kernel
 dpkg -l | grep pve-kernel
 
 # Check current running kernel 
 uname -r
 
 #Delete kernel old than current running kern
 apt purge pve-kernel-4.13.16-2-pve
 
 ```
 
 
## Error 2 :The configuration should be retried using`dpkg --configure <package>` or the configure menu option in dselect

When I met error like this.
 
### ERROR Detail: dpkg: error processing package pve-ha-manager (--configure): dependency problems - leaving triggers unprocessed Errors were encountered while processing:
 
 
 ``` shell
# $ apt update
#$ apt upgrade
#Reading package lists... Done
#Building dependency tree
#Reading state information... Done
#Calculating upgrade... Done
#0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
#7 not fully installed or removed.
#After this operation, 0 B of additional disk space will be used.
#Do you want to continue? [Y/n] y
#Setting up pve-cluster (5.0-30) ...
#Failed to retrieve unit: No such method 'GetUnit'
#Failed to retrieve unit: No such method 'GetUnit'
#Failed to start pve-ha-lrm.service: Unknown unit: pve-ha-lrm.service
#See system logs and 'systemctl status pve-ha-lrm.service' for details.
#dpkg: error processing package pve-cluster (--configure):
# subprocess installed post-installation script returned error exit status 1
#dpkg: dependency problems prevent configuration of pve-firewall:
# pve-firewall depends on pve-cluster; however:
#  Package pve-cluster is not configured yet.

#dpkg: error processing package pve-firewall (--configure):
# dependency problems - leaving unconfigured
#dpkg: dependency problems prevent configuration of libpve-guest-common-perl:
# libpve-guest-common-perl depends on pve-cluster; however:
#  Package pve-cluster is not configured yet.

#dpkg: error processing package libpve-guest-common-perl (--configure):
# dependency problems - leaving unconfigured
#dpkg: dependency problems prevent configuration of qemu-server:
# qemu-server depends on libpve-guest-common-perl; however:
#  Package libpve-guest-common-perl is not configured yet.
# qemu-server depends on pve-cluster; however:
#  Package pve-cluster is not configured yet.
# qemu-server depends on pve-firewall; however:
#  Package pve-firewall is not configured yet.

#dpkg: error processing package qemu-server (--configure):
# dependency problems - leaving unconfigured
#dpkg: dependency problems prevent configuration of libpve-storage-perl:
# libpve-storage-perl depends on pve-cluster; however:
#  Package pve-cluster is not configured yet.

#dpkg: error processing package libpve-storage-perl (--configure):
# dependency problems - leaving unconfigured
#dpkg: dependency problems prevent configuration of pve-manager:
# pve-manager depends on libpve-guest-common-perl (>= 2.0-14); however:
#  Package libpve-guest-common-perl is not configured yet.
# pve-manager depends on libpve-storage-perl (>= 5.0-18); however:
#  Package libpve-storage-perl is not configured yet.
# pve-manager depends on pve-cluster (>= 5.0-27); however:
#  Package pve-cluster is not configured yet.
# pve-manager depends on pve-firewall; however:
#  Package pve-firewall is not configured yet.
# pve-manager depends on qemu-server (>= 5.0-24); however:
#  Package qemu-server is not configured yet.

#dpkg: error processing package pve-manager (--configure):
# dependency problems - leaving unconfigured
#dpkg: dependency problems prevent configuration of pve-container:
# pve-container depends on libpve-guest-common-perl; however:
#  Package libpve-guest-common-perl is not configured yet.
# pve-container depends on libpve-storage-perl (>= 5.0-18); however:
#  Package libpve-storage-perl is not configured yet.
# pve-container depends on pve-cluster (>= 4.0-8); however:
#  Package pve-cluster is not configured yet.

#dpkg: error processing package pve-container (--configure):
# dependency problems - leaving unconfigured
#dpkg: dependency problems prevent processing triggers for pve-ha-manager:
# pve-ha-manager depends on pve-cluster (>= 3.0-17); however:
#  Package pve-cluster is not configured yet.
# pve-ha-manager depends on qemu-server; however:
#  Package qemu-server is not configured yet.

#dpkg: error processing package pve-ha-manager (--configure):
# dependency problems - leaving triggers unprocessed
#Errors were encountered while processing:
# pve-cluster
# pve-firewall
# libpve-guest-common-perl
# qemu-server
# libpve-storage-perl
# pve-manager
# pve-container
# pve-ha-manager
#E: Sub-process /usr/bin/dpkg returned an error code (1)
#
#$ dpkg -C
#The following packages have been unpacked but not yet configured.
#They must be configured using dpkg --configure or the configure
#menu option in dselect for them to work:
# libpve-guest-common-perl Proxmox VE common guest-related modules
# libpve-storage-perl  Proxmox VE storage management library
# pve-container        Proxmox VE Container management tool
# pve-firewall         Proxmox VE Firewall
# pve-manager          Proxmox Virtual Environment Management Tools
# qemu-server          Qemu Server Tools

#The following packages are only half configured, probably due to problems
#configuring them the first time.  The configuration should be retried using
#dpkg --configure <package> or the configure menu option in dselect:
# pve-cluster          Cluster Infrastructure for Proxmox Virtual Environment

#The following packages have been triggered, but the trigger processing
#has not yet been done.  Trigger processing can be requested using
#dselect or dpkg --configure --pending (or dpkg --triggers-only):
# pve-ha-manager       Proxmox VE HA Manager

 ```
 
### Solution:
 
 ``` shell
 dpkg --configure pve-manager
 dpkg --configure -a 
 
 #remove all of these software
 apt remove pve-cluster pve-ha-manager pve-container pve-manager libpve-storage-perl qemu-server libpve-guest-common-perl pve-firewall proxmox-ve
 
 

 
 ```
Then reinstall proxmox-ve
 
 
## Remove proxmox-ve

### Error `You are attempting to remove the meta-package 'proxmox-ve'!`

This is  proxmox ve warning. We need follow the instruction to uninstall proxmox ve.



``` shell
# **This is now command, this is prompt dialog**
Do you want to continue? [Y/n] y
W: (pve-apt-hook) !! WARNING !!
W: (pve-apt-hook) You are attempting to remove the meta-package 'proxmox-ve'!
W: (pve-apt-hook)
W: (pve-apt-hook) If you really you want to permanently remove 'proxmox-ve' from your system, run the following command
W: (pve-apt-hook)       touch '/please-remove-proxmox-ve'
W: (pve-apt-hook) and repeat your apt-get/apt invocation.
W: (pve-apt-hook)
W: (pve-apt-hook) If you are unsure why 'proxmox-ve' would be removed, please verify
W: (pve-apt-hook)       - your APT repository settings
W: (pve-apt-hook)       - that you are using 'apt-get dist-upgrade' or 'apt full-upgrade' to upgrade your system
E: Sub-process /usr/share/proxmox-ve/pve-apt-hook returned an error code (1)
E: Failure running script /usr/share/proxmox-ve/pve-apt-hook
```

### Solution
 
 ```shell
 touch '/please-remove-proxmox-ve'
 
 ```
 
## Error with update: /bin/sh: 1: /usr/share/proxmox-ve/pve-apt-hook: not found

### Error:
```shell
/bin/sh: 1: /usr/share/proxmox-ve/pve-apt-hook: not found
E: Sub-process /usr/share/proxmox-ve/pve-apt-hook returned an error code (127)
E: Failure running script /usr/share/proxmox-ve/pve-apt-hook
```

### Solution:

``` shell
mkdir /usr/share/proxmox-ve
touch /usr/share/proxmox-ve/pve-apt-hook
chmod  u+x /usr/share/proxmox-ve/pve-apt-hook
apt install proxmox-ve

```

 
 
## Swell
 
   - [Upgrade error](https://forum.proxmox.com/threads/upgrade-error.51034/)
   - [Installation completely broken](https://forum.proxmox.com/threads/installation-completely-broken.46796/)
   - [Error with update: /bin/sh: 1: /usr/share/proxmox-ve/pve-apt-hook: not found
](https://forum.proxmox.com/threads/error-with-update-bin-sh-1-usr-share-proxmox-ve-pve-apt-hook-not-found.45436/)