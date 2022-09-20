---
title: "Chang Volume of Docker From Local to Overlay2"
date: 2019-06-11T10:19:07+08:00
draft: true
---


# Change Volume format of docker in lxc from local to overlay2



# Loading module `overlay` in proxmox ve 5.x


```shell
#Load overlay module now
modprobe overlay
# Make sure overlay loaded with linux kernel
echo 'overlay' >> /etc/modules-load.d/modules.conf
```



# Change LXC settings

  - Login promxmox VE host.

 ``` shell
ssh root@10.32.0.23
```

 - Stop LXC  container which id is  `102`.

  ```shell
lxc-stop 102
```  
  - Change settings of the LXC container.

  ```shell
echo "#insert docker part below
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop:" >> /etc/pve/lxc/102.conf
```

  - Start lxc container.

  ```shell
lxc-start 102
```

# Change Docer Settings


  - Login LXC container

  ```shell
  # The ip address of the  lxc container  is 10.32.0.69
  ssh root@10.32.0.69
  
  ```
  
  - stop docker service

  ``` shell
  systemctl stop docker
  
  ```
  
  - change volume format of docker from local to overlay2

   ```shell
echo "{
    \"storage-driver\": \"overlay2\"
}" > /etc/docker/daemon.json
   ```
   
  - Start docker service
  
  ``` shell
  systemctl reload-daemon
  systemctl start docker
  
  ```
  
  - Chekc format of docker voluem

  ```shell
  docker info
  ```
  
  Here we go!
  
  ```shell
  root@xa-graylog3:~# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.09.6
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: false
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: bb71b10fd8f58240ca47fbb579b9d1028eea7c84
runc version: 2b18fe1d885ee5083ef9f0838fee39b62d653e30
init version: fec3683
Security Options:
 apparmor
 seccomp
  Profile: default
Kernel Version: 4.15.18-15-pve
Operating System: Ubuntu 18.04.2 LTS
OSType: linux
Architecture: x86_64
CPUs: 4
Total Memory: 8GiB
Name: xa-graylog3
ID: L6LU:FIDV:5QJP:36DL:L5HL:QXEN:FPML:HDUC:2TB4:6KC2:CEDY:COLI
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false
Product License: Community Engine
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```
  


