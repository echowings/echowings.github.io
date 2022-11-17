---
title: "Application Scheduling and Lifecycle Mangement"
date: 2022-11-17T15:47:56+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=0d188417
categories:
  - kubernetes
  - devop
tags:
  - kubernetes
  - k8s
  - devops
  - container
slug:
  - kubernetes
layout: 
  - post

license: CC BY-NC-ND
---

This article describes how to use kubernetes deployments to deploy pods, scale pods, perform rolling updates and rollbacks, carry out resource management, and use ConfigMaps to configurepods using `kubectl` command and YAML definitions.


## Run a container

### Cretae a pod using am imperative command
#### Create pods with `kubectl` command
```bash
kubectl run <pod-name> --image=<image-name:image-tag>
```
for instance:
```bash
kubectl run nginx-pod --image=nginx:alpine
```

#### Check the pod's status

```bash
kubectl describe pod nginx-pod
```

### Create pods with YAML-defined 
Create a yaml file

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
  image: nginx:alpine
  ports:
  - containerPort: 80
```

To create pods with kubectl apply -f file

```bash
kubectl apply -f <your-spec>.yaml
```

### run busybox 
```bash
kubctl run busybox --rm -it --image=busybox /bin/sh
```

### export yaml file with --dry-run

```bash
kubectl run nginx --image=nginx:alpine --dry-run -o yaml > nginx-sample.yaml
kubectl apply nginx-sample.yaml
```

## Understanding Liveness, readiness, and startup probes

To ex



