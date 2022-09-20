---
title: "How to Create Centos7 Template for Pve"
date: 2020-09-12T17:29:39+08:00
lastmode: 2020-09-12T17:29:39+08:00
draft: false
tags: [ "centos", "pve","linux" ]
categories: ["linux"]
reward: true
mathjax: true
---

# HOw to create CentOS 7 template for PVE

---


```bash

VMID=9002
VM_NAME=CentOS-7-x86_64-GenericCloud
VM_DOWNLOAD_URL=http://mirrors.ustc.edu.cn/centos-cloud/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2.xz
TEMPLATE_VM_NAME=$(echo $VM_NAME | sed 's/_/-/g')
STORAGE_PATH=local
DEFAULT_PVE_PATH=/var/lib/vz/images
#Change directory to `local`
cd /var/lib/vz/images

#Download centos 7 cloud images on proxmox ve
wget $VM_DOWNLOAD_URL

#unzip the file
#xz -z (to zip file)
#xz -k( to keep original file and valur `-0` to `-9` to set zip rate, default zip rate is -6
#xz -d (to unzip file)
# xz -k (to keep zipped file not deleted)
xz -d -k $VM_NAME.qcow2.xz

qm create $VMID --memory 4096 --net0 virtio,bridge=vmbr0 --name $TEMPLATE_VM_NAME

qm importdisk $VM_ID $VM_NAME.qcow2 $STORAGE_PATH

qm set $VM_ID --scsihw virtio-scsi-pci --scsi0 $STORAGE_PATH:$VM_ID/vm-$VM_ID-disk-0.raw

qm set $VM_ID --scsihw virtio-scsi-pci --scsi0 $STORAGE_PATH:$VM_ID/vm-$VM_ID-disk-0.raw

qm set $VM_ID --boot c --bootdisk scsi0
qm set $VM_ID --serial0 socket --vga serial0
qm set $VM_ID --agent 1

qemu-img convert -f raw $DEFAULT_PVE_PATH/$VM_ID/vm-$VM_ID-disk-0.raw -O qcow2 $DEFAULT_PVE_PATH/$VM_ID/vm-$VM_ID-disk-0.qcow2


sed -i 's/.raw/.qcow2/g' /etc/pve/qemu-server/$VM_ID.conf
qm start $VM_ID
qm terminal $VM_ID



#Change yum source form offical to mirrors.ustc.edu.cn
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Base.repo
         
yum makecache
yum update
yum upgrade






```