---
title: "How to install kubernetes on ubuntu 20.04"
date: 2022-11-13T18:53:31+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=dddca1a9
categories:
  - Blog
tags:
  - devops
  - kubernetes
  - k8s
  - ubuntu
  - sre
slug:
  - test
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

# How to install kubernetes on ubuntu 20.04
I have 3 proxmox ve node separately. So here are the steps:

  1. Create ubuntu 20.04 cloud image on 3 proxmox ve server
  2. Delpy 3 ubuntu 20.04 virtual machines on each 3 proxmox ve server.(1 for kubernetes master node, other 2 for kubernetes work nodes)
  3. Install kubeadm,kubelet,kubectl,containerd on each nodes(includ master node and worker nodes)
  4. Initalize Kubernetes on master node and install calico network CNI
  5. Let worker nodes join kubernets cluster
  

## Provision 3 ubuntu server  on 3 proxmox ve nodes
### Create ubuntu 20.04 template on 3 proxmox ve nodes
```bash
cat << 'EOF' | tee create_ubuntu_20.04_template.sh
#!/bin/bash
VMID=9003
IMAGE_NAME="focal-server-cloudimg-amd64.img"
VM_NAME="ubuntu-20.04TLS-template"
STORAGE="SSD"
STORAGE_FULL_PATH="/mnt/pve"
VCPUS=4
VM_MEMORY=4096
#STORAGE_FULL_PATH="/var/lib/vz"

qm create $VMID --name ${VM_NAME} \
	--memory ${VM_MEMORY}  \
	--net0 virtio,bridge=vmbr0 \
	--cores ${VCPUS} \
	--sockets 1 \
	--cpu cputype=host \
	--description "${VM_NAME}" \
	--kvm 1 \
	--machine q35

qm importdisk $VMID /var/lib/vz/template/qcow/${IMAGE_NAME} ${STORAGE}
qm set $VMID --scsihw virtio-scsi-pci --virtio0 ${STORAGE}:${VMID}/vm-$VMID-disk-0.raw
qm set $VMID --serial0 socket
qm set $VMID --boot c --bootdisk virtio0
qm set $VMID --agent 1
qm set $VMID --hotplug disk,network,usb
qm set $VMID --vcpus ${VCPUS}
qm set $VMID --vga qxl
qm set $VMID --name ${VM_NAME}
qm set $VMID --ide2 ${STORAGE}:cloudinit
qm set $VMID --serial0 socket --vga serial0
qm set $VMID --sshkey /etc/pve/pub_keys/pub_key.pub
qm set $VMID --ciuser ubuntu
qm set $VMID --cipassword xA123456
qm set $VMID --ipconfig0 ip=dhcp
qm resize $VMID virtio0 +50G


qemu-img convert -f raw ${STORAGE_FULL_PATH}/${STORAGE}/images/$VMID/vm-$VMID-disk-0.raw  -O qcow2 ${STORAGE_FULL_PATH}/${STORAGE}/images/$VMID/vm-$VMID-disk-0.qcow2

sed -i 's/.raw/.qcow2/g' /etc/pve/qemu-server/${VMID}.conf

rm -rf ${STORAGE_FULL_PATH}/${STORAGE}/images/${VMID}/*.raw
qm set $VMID --template
EOF
# Run script to create ubuntu 20.04 template
bash create_ubuntu_20.04_template.sh

```

# 1.2 Create 3 kubernetes nodes from template 
```bash

qm clone 9003 113 --full --name ubuntu-k8s-master
qm set 113 --ipconfig0 ip=192.168.11.71/24,gw=192.168.11.1
qm set 113 --onboot 1

qm clone 9003 113 --full --name ubuntu-k8s-node01
qm set 113 --ipconfig0 ip=192.168.11.72/24,gw=192.168.11.1
qm set 113 --onboot 1

qm clone 9003 113 --full --name ubuntu-k8s-node02
qm set 113 --ipconfig0 ip=192.168.11.73/24,gw=192.168.11.1
qm set 113 --onboot 1
```


## Install Prerequest packects
###  Change ubuntu sourcelist and update OS if nessary

```bash 
source /etc/os-release
echo $VERSION_CODENAME
[ ! -f /etc/apt/sources.list.bak ] &&mv /etc/apt/sources.list{,.bak}
[ ! -f /etc/apt/sources.list ] &&curl -fsSL https://mirrors.ustc.edu.cn/repogen/conf/ubuntu-https-4-${VERSION_CODENAME} -o /etc/apt/sources.list
sudo apt-get update
sudo apt -y full-upgrade
[ -f /var/run/reboot-requried ] && sudo reboot -f
```

### Forwarding IPv4 and letting iptables see bridged traffic

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Reload sysctl
sudo sysctl --system
```

### Install containerd runtime
```bash
sudo apt-get update
#sudo apt-get install  ca-certificates  curl  gnupg lsb-release
# Install required packages
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates lsb-release

# Add Docker's offical GPG key
sudo mkdir -p /etc/apt/keyrings
[ ! -f /etc/apt/keyrings/docker.gpg ] &&curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

[ ! -f /etc/apt/sources.list.d/docker.list ] &&echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null



# Install
sudo apt update
sudo apt install -y containerd.io
```
### Modify containerd configuration

```bash
[ ! -f /etc/containerd ] && mkdir -p /etc/containerd
sudo rm -rf /etc/containerd/config.toml
sudo containerd config default | sudo tee /etc/containerd/config.toml

#set plugins.cri.systemd_cgroup = true in /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup\ =\ false/SystemdCgroup\ =\ true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Disable SWAP


```bash
sudo sed -i '/swap/d' /etc/fstab
# Search for a swap line and add # (hashtag) sign in front of the line.
# sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
sudo mount -a
free -h

```

### Install kubectl，kubeadm,kubelet
```bash
sudo apt-get install -y ca-certificates curl apt-transport-https vim git curl wget 

# if you use Debian 9(stretch) or earlier you would also need to install `apt-transport-https`
sudo apt-get install -y apt-transport-https

# Download the Google Cloud public signing key
[ ! -f /usr/share/keyrings/kubernetes-archive-keyring.gpg ]&&sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add the kubernetes apt repository
[ ! -f /etc/apt/sources.list.d/kubernetes.list ] && echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt update
apt-cache madison kubeadm

#1.23.10-00
sudo apt-get install -y kubelet=1.23.10-00 kubeadm=1.23.10-00 kubectl=1.23.10-00
# Provent auto update new version
sudo apt-mark hold kubelet kubeadm kubectl

# Verify whether kubectl has been successfully installed by running the following command upon the completion of the previous steps: 
kubectl version --client
kubeadm version
kubelet --version
```

## Setup master node
###  Inital Kubernetes master node

```bash
sudo kubeadm config images pull
 
kubeadm init --pod-network-cidr=10.244.0.0/16 --upload-certs

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install calico 
```bash

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
# Monitor pod status
watch kubectl get pods -n calico-system
kubectl get pods -n calico-system -w

```

###  Option: Single kubernetes nodes to remove taint on master nodes
```bash
# Single node k8s 
#kubectl taint nodes --all node-role.kubernetes.io/master-
#kubectl taint nodes --all  node-role.kubernetes.io/control-plane-

```

### Option: query join command

```bash
# if you forget the join information, you can query it with commands as show below
kubeadm token create  --print-join-command
```
## Setup Worker Node

```bash
sudo kubeadm config images pull

kubeadm join 192.168.11.71:6443 --token akn012.rp0e7oxw0qn7b5o3 \
	--discovery-token-ca-cert-hash sha256:f78548da4af356ea8b006531962673b5945dd1c36588e137c6ec44c99d4ad7e1
```

## Check Node status

```bash
kubectl get nodes
kubectl get pods -A -o wide
```

## Reference


  1. [Deploy metrics-server](https://skyao.io/learning-kubernetes/docs/installation/kubeadm/metrics-server.html)

  2. [Deploy kubevirt](https://blog.csdn.net/weixin_45804031/article/details/124783723)

  3. [Kubernetes Kubevirt ](https://www.infvie.com/ops-notes/kubernetes-kubevirt-virtual-machine-management.html)

  4. [DEPLOY A KUBERNETES CLUSTER USING ANSIBLE](https://buildvirtual.net/deploy-a-kubernetes-cluster-using-ansible/)
  5. [Install Kubernetes Cluster on Ubuntu 20.04 with kubeadm](https://computingforgeeks.com/deploy-kubernetes-cluster-on-ubuntu-with-kubeadm/)





