---
title: "How to Create Single Kubernetes Cluster With Talos on Bare Metal"
date: 2023-12-26T15:31:54+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=24352ced
categories:
  - Blog
tags:
  - Hugo
slug:
  - test
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---




## Install talosctl on your laptop or client
### Install talosctl on your system


```shell
curl -sL https://talos.dev/install | sh
```


## Setup the Talos Linux Nodes

### Download latest talos iso
```shell
##For amd64
VER=$(curl -s https://api.github.com/repos/siderolabs/talos/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
echo $VER
wget https://github.com/siderolabs/talos/releases/download/v${VER}/talos-amd64.iso

##For RM64
wget https://github.com/siderolabs/talos/releases/download/v${VER}/talos-arm64.iso
```

### Boot talos iso on bare metal server
write talos on usb and boot it on usb live mode

Then setup the static ip with `F3`
|Key| Value|
|---|---|
|Hostname | talos-k8s |
|DNS Servers|192.168.11.10|
|Time Servers|0.pool.ntp.org |
|Interface|enp1s0|
|Mode| Static|
|Addresses| 192.168.11.61/24|
|Gateway| 192.168.11.1|

## setup talos


### Delete exist configure

```shell
 rm -rf secrets.yaml worker.yaml controlplane.yaml talosconfig   ~/.talos ~/_out ~/.kube
```

### Generate configure

```shell
CONTROL_PLANE_IP="192.168.11.61"
talosctl gen secrets -o secrets.yaml 
talosctl gen config --with-secrets secrets.yaml my-cluster https://$CONTROL_PLANE_IP:6443 \
    --output-dir _out 
 talosctl -n $CONTROL_PLANE_IP disks --insecure

 #modify disk
#/dev/mmcblk0
sed -i 's/sda/mmcblk0/g' _out/controlplane.yaml _out/worker.yaml 


 sed -i 's/\#\ allowSchedulingOnControlPlanes\:\ true/allowSchedulingOnControlPlanes\:\ true/g' _out/worker.yaml 
sed -i 's/^\s*# allowSchedulingOnControlPlanes: true/    allowSchedulingOnControlPlanes: true/' _out/controlplane.yaml
 #Save the file and fire up the control plane first:
talosctl apply-config --insecure -n $CONTROL_PLANE_IP --file _out/controlplane.yaml

# 2. Run the Worker Nodes
talosctl apply-config --insecure -n $CONTROL_PLANE_IP --file _out/worker.yaml


# 3. Bootstrap Etcd
# export CONTROL_PLANE_IP=$TALOS_IP
export TALOSCONFIG="_out/talosconfig"
talosctl config endpoint $CONTROL_PLANE_IP
talosctl config node $CONTROL_PLANE_IP

#Now we will set the endpoints and nodes.
talosctl --talosconfig _out/talosconfig config endpoint $CONTROL_PLANE_IP
talosctl --talosconfig _out/talosconfig config node $CONTROL_PLANE_IP

#Bootstrap etcd
talosctl --talosconfig _out/talosconfig bootstrap -n $CONTROL_PLANE_IP


#4. Access Talos Powered Kubernetes Cluster
#Once the cluster is up, you can access and use it as desired to run the containerized workloads. But first, obtain the admin kubeconfig
talosctl --talosconfig _out/talosconfig kubeconfig .


curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin

mkdir -p $HOME/.kube
sudo cp -i kubeconfig $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# View the nodes in the cluster
kubectl get nodes -o wide

# View the pods:
kubectl get pods -A


5. Deploy a Test Application on Kubernetes
To verify if the cluster is working properly, we can deploy a sample Nginx application. To achieve that, we can use the below manifest:

kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
EOF


#View if the pods are running:
kubectl get pods


#Expose the app with NodePort:
kubectl expose deployment nginx-deployment --type=NodePort --port=80

# Get the service port:
kubectl get svc


```

## Reference

  - [Install Kubernetes Cluster using Talos Container Linux](https://computingforgeeks.com/install-kubernetes-using-talos-container-linux/)
  - [How to enable workers on your control plane nodes](https://www.talos.dev/v1.6/talos-guides/howto/workers-on-controlplane/)

