---
title: "How to Create Vyos Template on Proxmox"
date: 2024-02-21T17:24:06+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=16de454a
categories:
  - pve
  - vyos
tags:
  - vyos
  - pve
  - proxmox
  - proxmox ve
  - template
slug:
  - pve
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---



```shell
#!/bin/bash
set -x
#Create template
#args:
# vm_id
# vm_name
# file name in the current directory
function create_template() {
    #Print all of the configuration
    echo "Creating template $2 ($1)"
    echo "------------"
    echo "$1"
    echo "$2"
    echo "$3"
    echo "$4"
    echo "$5"
    echo "------------"
    if [ $(qm list | grep $1 | wc -l) -eq 1 ]
    then
      echo "VM already  exist"
      echo "Will delete the vm"
      qm destroy $1
      if [ $? -eq 0 ]
      then
          echo "delete vm successfully!"
      else
          echo "delete failed!"
      fi
    else
      echo  "VM didn't exist"
      echo  "will create it"
    fi
    #Create new VM
    #Feel free to change any of these to your liking
    qm create $1 --name $2 --ostype l26
    #Set networking to default bridge
    qm set $1 --net0 virtio,bridge=vmbr0
    #Set display to serial
    qm set $1 --serial0 socket --vga serial0
    #Set memory, cpu, type defaults
    #If you are in a cluster, you might need to change cpu type
    qm set $1 --memory 1024 --cores 4 --cpu host
    #Set boot device to new file
    #qm set $1 --scsi0 ${storage}:0,import-from="$(pwd)/$3",discard=on
    qm set $1 --scsi0 ${storage}:0,import-from="$5/$3",discard=on
    #Set scsi hardware as default boot disk using virtio scsi single
    qm set $1 --boot order=scsi0 --scsihw virtio-scsi-single
    #Enable Qemu guest agent in case the guest has it available
    qm set $1 --agent enabled=1,fstrim_cloned_disks=1
    #Add cloud-init device
    qm set $1 --ide2 ${storage}:cloudinit
    #Set CI ip config
    #IP6 = auto means SLAAC (a reliable default with no bad effects on non-IPv6 networks)
    #IP = DHCP means what it says, so leave that out entirely on non-IPv4 networks to avoid DHCP delays
    qm set $1 --ipconfig0 "ip6=auto,ip=dhcp"
    #Import the ssh keyfile
    qm set $1 --sshkeys ${ssh_keyfile}
    #If you want to do password-based auth instaed
    #Then use this option and comment out the line above
    #qm set $1 --cipassword password
    #Add the user
    qm set $1 --ciuser ${username}
    #Resize the disk to 8G, a reasonable minimum. You can expand it more later.
    #If the disk is already bigger than 8G, this will fail, and that is okay.
    #qm disk resize $1 scsi0 8G

    #Convert raw disk to qcow2
    VM_FOLDER=$4
    qemu-img convert -f raw $VM_FOLDER/$1/vm-$1-disk-0.raw -O qcow2 $VM_FOLDER/$1/vm-$1-disk-0.qcow2
    rm -rf $VM_FOLDER/$1/vm-$1-disk-0.raw
    sed -i 's/.raw/.qcow2/g' /etc/pve/qemu-server/$1.conf

    #Make it a template
    qm template $1



    #Remove file when done
    rm $3


}





#Path to your ssh authorized_keys file
#Alternatively, use /etc/pve/priv/authorized_keys if you are already authorized
#on the Proxmox system
export ssh_keyfile=/root/id_rsa.pub
#Username to create on VM template
export username=echowings

#Name of your storage
export storage=SSD

export ID="9001"
export TEMPLATE_VM_NAME="template-vyos-1.3.6"
export VYOS_QEMU_IMAGE="vyos-1.3.6-cloud-init-2G-qemu.qcow2"
#export VM_IMAGE_FOLDER="/var/lib/vz/template"
export QCOW2_IMAGE_FOLDER="/var/lib/vz/template"
export VM_IMAGE_FOLDER="/mnt/pve/SSD/images"
export VM_DOWNLOAD_URL="https://xxxxx.xxxx.xxx/vyos-1.3.6-cloud-init-2G-qemu.qcow2"

tee $ssh_keyfile << "EOF"
ssh-rsa $YOURKEY
EOF

ls -al $QCOW2_IMAGE_FOLDER/$VYOS_QEMU_IMAGE
if [ ! -f $QCOW2_IMAGE_FOLDER/$VYOS_QEMU_IMAGE ]
then
        wget  $VM_DOWNLOAD_URL -O $QCOW2_IMAGE_FOLDER/$VYOS_QEMU_IMAGE
fi
cd $VM_IMAGE_FOLDER
create_template $ID $TEMPLATE_VM_NAME $VYOS_QEMU_IMAGE $VM_IMAGE_FOLDER $QCOW2_IMAGE_FOLDER

```

