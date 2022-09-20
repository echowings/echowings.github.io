---
title: "Install Kubernetes on CentOS7"
date: 2019-05-01T19:14:51+08:00
draft: flase
tags: ["k8s", "devops", "centos"]
categories: ["devops"]
auther: "steve dong"
reward: true

---


# Install Kubernetes on Centos 7

## Disable and stop firewall

```shell
# Disable firewall service
systemctl disable firewalld

# Stop firewall service
systemctl stop firewalld

```

## Install etcd and kubernetes
Install etcd and kubernetes and it will also install docker.

``` shell
yum install -y etcd kubernetes

```

## Start service in order

``` shell
systemctl start etcd
systemctl start docker
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler
systemctl start kubelet
systemctl start start kube-proxy

```


