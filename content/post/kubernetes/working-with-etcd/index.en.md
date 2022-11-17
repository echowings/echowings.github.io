---
title: "Working With Etcd"
date: 2022-11-15T18:32:33+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=78542a6a
categories:
  - kubernetes
  - devops
tags:
  - kubernetes
  - devops
  - etcd
slug:
  - k8s
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---
# Working with Etcd
Today, we will learn how to interactve with etcd . to backup and restore the etcd database.
First thing first, let's check the etcd pod name in our kubernetes cluster. Let's go!
## Check etcd pod name

Run the command on master node to get the etcd pod name
```bash
kubectl get pods -n kube-system
```
output:

```bash
root@ubuntu-k8s-master:/home/ubuntu# kubectl get pods -n kube-system
NAME                                        READY   STATUS    RESTARTS        AGE
calico-kube-controllers-7b8458594b-5d84q    1/1     Running   86 (104m ago)   26h
calico-node-2l8mt                           1/1     Running   2 (104m ago)    2d17h
calico-node-hrfjc                           1/1     Running   2 (105m ago)    2d17h
calico-node-rkwth                           1/1     Running   5 (105m ago)    2d17h
coredns-64897985d-cb85v                     1/1     Running   1 (104m ago)    26h
coredns-64897985d-fp25w                     1/1     Running   1 (105m ago)    26h
etcd-ubuntu-k8s-master                      1/1     Running   73 (17m ago)    2d18h
kube-apiserver-ubuntu-k8s-master            1/1     Running   63 (15m ago)    26h
kube-controller-manager-ubuntu-k8s-master   1/1     Running   8 (105m ago)    26h
kube-proxy-5xrhj                            1/1     Running   2 (105m ago)    26h
kube-proxy-qqkbz                            1/1     Running   1 (105m ago)    26h
kube-proxy-t7l2p                            1/1     Running   1 (104m ago)    26h
kube-scheduler-ubuntu-k8s-master            1/1     Running   7 (105m ago)    26h
metrics-server-7cf8b65d65-wnvtd             1/1     Running   2 (105m ago)    2d17h
```
My single kubernetes cluster has etcd pod named `etcd-ubuntu-k8s-master`


## Exporing the ETCD cluster pod

```bash
# kubectl describe po <etcd-podnam> -n kube-system
kubectl describe po etcd-ubuntu-k8s-master
```

output:

```bash
root@ubuntu-k8s-master:/home/ubuntu# kubectl describe po etcd-ubuntu-k8s-master -n kube-system
Name:                 etcd-ubuntu-k8s-master
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 ubuntu-k8s-master/192.168.11.71
Start Time:           Mon, 14 Nov 2022 07:54:51 +0000
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.11.71:2379
                      kubernetes.io/config.hash: 0855baf22e98bdcfb39a80e74a5324cb
                      kubernetes.io/config.mirror: 0855baf22e98bdcfb39a80e74a5324cb
                      kubernetes.io/config.seen: 2022-11-12T16:24:44.452715208Z
                      kubernetes.io/config.source: file
                      seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:               Running
IP:                   192.168.11.71
IPs:
  IP:           192.168.11.71
Controlled By:  Node/ubuntu-k8s-master
Containers:
  etcd:
    Container ID:  containerd://65bbf08b85d61fe8ab3fef0224bce17dd48997fb59d6f2bc23a638ba729d00e0
    Image:         k8s.gcr.io/etcd:3.5.1-0
    Image ID:      k8s.gcr.io/etcd@sha256:64b9ea357325d5db9f8a723dcf503b5a449177b17ac87d69481e126bb724c263
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.11.71:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --experimental-initial-corrupt-check=true
      --initial-advertise-peer-urls=https://192.168.11.71:2380
      --initial-cluster=ubuntu-k8s-master=https://192.168.11.71:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.11.71:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.11.71:2380
      --name=ubuntu-k8s-master
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Tue, 15 Nov 2022 10:22:56 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    2
      Started:      Tue, 15 Nov 2022 10:17:48 +0000
      Finished:     Tue, 15 Nov 2022 10:17:48 +0000
    Ready:          True
    Restart Count:  73
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type    Reason   Age   From     Message
  ----    ------   ----  ----     -------
  Normal  Pulled   16m   kubelet  Container image "k8s.gcr.io/etcd:3.5.1-0" already present on machine
  Normal  Created  16m   kubelet  Created container etcd
  Normal  Started  16m   kubelet  Started container etcd
```

We need remember 

```bash
# etcd data directory
--data-dir=/var/lib/etcd

# Etcd endpoint
--listen-client-urls=https://127.0.0.1:2379,https://192.168.11.71:2379
```

## Listing etcd cluster members

```bash
kubectl exec etcd-ubuntu-k8s-master -n kube-system \
	-- sh -c "ETCDCTL_API=3 etcdctl  \
	--endpoints=https://127.0.0.1:2379 \
	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
	--cert=/etc/kubernetes/pki/etcd/server.crt \
	--key=/etc/kubernetes/pki/etcd/server.key \
	--write-out=table \
	member list"
```

output:

```bash
+------------------+---------+-------------------+-----------------------+----------------------------+------------+
|        ID        | STATUS  |       NAME        |      PEER ADDRS       |        CLIENT ADDRS        | IS LEARNER |
+------------------+---------+-------------------+-----------------------+----------------------------+------------+
| 8e9e05c52164694d | started | ubuntu-k8s-master | http://localhost:2380 | https://192.168.11.71:2379 |      false |
+------------------+---------+-------------------+-----------------------+----------------------------+------------+
```

## Checking the etcd cluster status
`ETCDCTL_API=3 etcdctl endpoint status`

```bash
kubectl -n kube-system exec etcd-ubuntu-k8s-master \
	-- sh  -c "ETCDCTL_API=3 etcdctl  \
	--endpoints=https://192.168.11.71:2379 \
	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
	--cert=/etc/kubernetes/pki/etcd/server.crt \
	--key=/etc/kubernetes/pki/etcd/server.key \
	--write-out=table \
	endpoint status"
```
output:

```bash
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|          ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| https://192.168.11.71:2379 | 8e9e05c52164694d |   3.5.1 |   10 MB |      true |      false |         2 |      32504 |              32504 |        |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

```bash
kubectl -n kube-system exec etcd-ubuntu-k8s-master \
	-- sh  -c "ETCDCTL_API=3 etcdctl \
	--endpoints=https://192.168.11.71:2379 \
	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
	--cert=/etc/kubernetes/pki/etcd/server.crt\
	--key=/etc/kubernetes/pki/etcd/server.key \
	--write-out=table \
	endpoint health "
```
output:

```bash
+----------------------------+--------+-------------+-------+
|          ENDPOINT          | HEALTH |    TOOK     | ERROR |
+----------------------------+--------+-------------+-------+
| https://192.168.11.71:2379 |   true | 14.261475ms |       |
+----------------------------+--------+-------------+-------+
```

## Installing etcd

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.5.5/etcd-v3.5.5-linux-amd64.tar.gz
tar -zxvf etcd-v3.5.5-linux-amd64.tar.gz
sudo mv etcd-v3.5.5-linux-amd64/etcd* /usr/local/bin
etcdctl version
```
or
```bash
kubectl -n kube-system exec etcd-ubuntu-k8s-master \
	-- sh -c "ETCDCTL_API=3 etcdctl version"
```

## Backing up etcd

### SSH to the etcd cluster node.
--skip--
### check out etcd status

```bash
# kubectl describe <etcd-podname> 
kubectl describe etcd-ubuntu-k8s-master 
kubectl -n kube-system exec etcd-ubuntu-k8s-master \
	-- sh -c "ETCDCTL_API=3 etcdctl \
	--endpoints=https://192.168.11.71:2379 \
	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
	--cert=/etc/kubernetes/pki/etcd/server.crt \
	--key=/etc/kubernetes/pki/etcd/server.key \
	--write-out=table endpoint status"

```
### Perform the etcd backup
```bash
sudo kubectl -n kube-system exec etcd-ubuntu-k8s-master. \
 	-- sh -c "ETCDCTL_API=3 etcdctl \
 	--endpoints=https://192.168.11.71:2379 \
 	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
 	--cert=/etc/kubernetes/pki/etcd/server.crt \
 	--key=/etc/kubernetes/pki/etcd/server.key \
 	snapshot save"
```
output

```bash
snapshotdb"{"level":"info","ts":1668525609.0965416,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"snapshotdb.part"}
{"level":"info","ts":1668525609.1151485,"logger":"client","caller":"v3/maintenance.go:211","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1668525609.115235,"caller":"snapshot/v3_snapshot.go:76","msg":"fetching snapshot","endpoint":"https://192.168.11.71:2379"}
{"level":"info","ts":1668525609.3567731,"logger":"client","caller":"v3/maintenance.go:219","msg":"completed snapshot read; closing"}
{"level":"info","ts":1668525609.393935,"caller":"snapshot/v3_snapshot.go:91","msg":"fetched snapshot","endpoi:nt":"https://192.168.11.71:2379","size":"10 MB","took":"now"}
{"level":"info","ts":1668525609.3941526,"caller":"snapshot/v3_snapshot.go:100","msg":"saved","path":"snapshotdb"}
```

### Exit the master nodes

## Restoring etcd

### SSH to the etcd cluster node.

-- skip --
### Check the etcd status

```bash
sudo ETCDCTL_API=3 etcdctl  \
	--endpoint https://192.168.11.71:2379 \  
	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
	--cert=/etc/kubernetes/pki/etcd/server.crt \
	--key=/etc/kubernetes/pki/etcd/server.key  \
	restore snapshotdb
```

### Restore the etcd backup

```bash
sudo ETCDCTL_API=3 etcdctl --endpoint https://192.168.11.71:2379 
	--cacert=/etc/kubernetes/pki/etcd/ca.crt \
	--cert=/etc/kubernetes/pki/etcd/server.crt \
	--key=/etc/kubernetes/pki/etcd/server.key \
	--write-out=table \
	endpoint status
```

### Exit the master node

--skip--




