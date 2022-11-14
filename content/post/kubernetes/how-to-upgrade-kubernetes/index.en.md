---
title: "How to Upgrade Kubernetes"
date: 2022-11-14T15:19:53+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=50442c4a
categories:
  - devops
tags:
  - kubrenetes
  - cka
  - devops
slug:
  - kubernetes
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


# How to upgrade kubernetes
To upgrade kuberetes I will 2 steps to implement it.
## Master node upgrade process
### Check out the current kubeadm and kubelet version
First thing first, login the master node with ssh.

```bash
kubeadm version
kubectl --version

## Running result
kubelet --version
Kubernetes v1.23.10
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.10", GitCommit:"7e54d50d3012cf3389e43b096ba35300f36e0817", GitTreeState:"clean", BuildDate:"2022-08-17T18:31:47Z", GoVersion:"go1.17.13", Compiler:"gc", Platform:"linux/amd64"}
```
### Install the new kubeadm version
We will upgrade kubernetes from v1.23.10-00 to  1.23.11-00
```bash
sudo su

# Update apt list
apt update
# Check the latest version available 
apt-cache madison kubeadm

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.11-00 && \
apt-mark hold kubeadm

# Check kubeadm version
kubeadm version
```


### Plan the kubeadm upgrade
```bash
kubeadm upgrade plan
```

### Apply the kubeadm upgrade

```bash
kubeadm upgrade apply v1.23.11
```

### Cordon the node

```bash
# check nodes name
kubectl get nodes
# drain the master nodes named 'ubuntu-k8s-master'
kubectl drain ubuntu-k8s-master --ignore-daemonsets


```

### Upgrade the kubelet and kubectl
```bash
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.23.11-00 kubectl=1.23.11-00 && \
apt-mark hold kubelet kubectl

# Restart the kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Uncordon the node
```bash
kubectl uncordon ubuntu-k8s-master
```


## Upgrade Worker Nodes

### Check out the current kubeadm and kubelet version

```bash
kubeadm version
kubelet version
```


### Install the new kubeadm version

```bash
sudo su
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.11-00 && \
apt-mark hold kubeadm
```

### Plan the kubeadm upgrade

```bash 
kubeadm upgrade node
```

### Apply the kubeadm upgrade

```bash
apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.23.11-00 kubectl=1.23.11-00 && \
apt-mark hold kubelet kubectl

# Restart the kubelet for the changes to take effect:
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Cordon the node

### Upgrade the kubelet

### uncordon the node

### Exit the worker node

