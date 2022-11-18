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

This article describs how to use kubernetes deployments to deploy pods, scale pods, perform rolling updates and rollbacks, carry out resource management, and use ConfigMaps to configurepods using `kubectl` command and YAML definitions.


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

### Create pods with declarative management
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

#### run busybox 
```bash
kubctl run busybox --rm -it --image=busybox /bin/sh
```

#### export yaml file with --dry-run

```bash
kubectl run nginx --image=nginx:alpine --dry-run -o yaml > nginx-sample.yaml
kubectl apply nginx-sample.yaml
```

## Understanding Liveness, readiness, and startup probes
  - Liveness probes
  - Readiness probes
  - Startup probes

## Understanding a multi-conainter pod

Multi-container pods are simply pods with more than one container working together as a single unit. When it coms to multiple containers residing in a pod, a conainer ineracts with another in the following two ways:

  - **Shared networking:** When two containers are running on the same host when they are in the same pod, they can access each other by simpley using localhost. All the listening ports are accessible to other containers in the pod, even if they're not exposed outside the pod.
 
  - **Shared storage volumes:** We can mount the same volume to two different conainers so that they can both interact with the same data - it is possible to have one container write data to the volume and the other container read that data from the same volume. Some volumes even allow concurrent reading and writing. 

### Create multiple containers in a pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  neme: multi-app-pod
  labels:
    app: multi-app
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80"wq
    - 
  - name: busybox-sidecar
    image: busybox
    command: [ 'sh', '-c', 'while true; do sleep 3600; done;' ]
    
```

## Understanding an init container

```yaml
apiVerion: v1
kind: Pod
metedata:
  name: melon-pod
  labels:
  	app: melonapp
spec:
  containers:
  - name: melonapp-container
    image:busybox:latest
    command: ['sh', '-c', 'echo The melonapp is running! && sleep 3600']
  initContainers:
  - name: init-melonservice
    image: busybox:latest
    command: ['sh', '-c', 'until nslookup melonservice; do echo waiting for melonserivce; sleep 2; done;']
  - name: init-melondb
    image: busybox:latest
    command: ['sh', '-c', 'until nslookup melondb; do echo waiting for melondb; sleep 2; done;']
```

