---
title: "How to Create Single Kubernetes Cluster With Talos on Bare Metal"
date: 2023-12-26T15:31:54+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=24352ced
categories:
  - cloud native
  - k8s
  - kvm
  - virtualization
tags:
  - talos
  - k8s
  - bare metal
  - kubevirt
  - kvm
slug:
  - k8s
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

Talos, a ligthweight OS for kubernetes, enables unified management of VMs and containers when paired with Kubevirt.

From today, I will try to build a network router gateway with talos , kubevirt and VyOS.

## Install talosctl on your laptop or client
### Install talosctl on your system


```shell
# Install on macos
brew install siderolabs/tap/talosctl

# Install talosctl on linux
curl -sL https://talos.dev/install | sh

## Manual install on linux
VER=$(curl -s https://api.github.com/repos/siderolabs/talos/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
echo $VER
curl -fsSL https://github.com/siderolabs/talos/releases/download/v${VER}/talosctl-linux-amd64 -o /usr/local/bin/talosctl
chmod +x /usr/local/bin/talosctl
/usr/local/bin/talosctl version

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
>> If you met loop reboot, try to install drivers with customized talos image.
For instance, If you want to install talos on N100 mini PC with network adapter intel v226.

 1. Acce [https://factory.talos.dev/](https://factory.talos.dev/)
 2. Select `Bare-metal Machine`, then click `Next`;
 3. Choose Talos Linux Version, then next.
 4. Choose Machine Archiitecture. For N100 I select `amd64`.
 5. In `System Extensions`, Qury `intel`. then select `i915` and `mei`.
 6. Downlos iso file from [Talos Linux Image Factory](https://factory.talos.dev/?arch=amd64&board=undefined&cmdline-set=true&extensions=-&extensions=siderolabs%2Fi915&extensions=siderolabs%2Fintel-ice-firmware&extensions=siderolabs%2Fintel-ucode&extensions=siderolabs%2Fmei&platform=metal&secureboot=undefined&target=metal&version=1.9.0)


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
|Addresses| 192.168.11.50/24|
|Gateway| 192.168.11.1|

## setup talos


### Delete exist configure
Delete config files.
```shell
 rm -rf secrets.yaml worker.yaml controlplane.yaml talosconfig   ~/.talos ~/_out ~/.kube
```

### Generate configure

```shell
CONTROL_PLANE_IP="192.168.11.50"
CLUSTER_NAME="myhome-talos-cluster"
talosctl gen secrets -o secrets.yaml 
talosctl gen config --with-secrets secrets.yaml $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443 \
    --output-dir _out 
 #talosctl -n $CONTROL_PLANE_IP disks --insecure
 talosctl -n $CONTROL_PLANE_IP get disks --insecure


 #modify disk
#/dev/mmcblk0
#sed -i 's/sda/mmcblk0/g' _out/controlplane.yaml _out/worker.yaml 
sed -i 's/sda/nvme0n1/g' _out/controlplane.yaml _out/worker.yaml 


#sed -i 's/\#\ allowSchedulingOnControlPlanes\:\ true/allowSchedulingOnControlPlanes\:\ true/g' _out/worker.yaml 

# vi _out/controlplane.yaml
#     nodeLabels:
#         node.kubernetes.io/exclude-from-external-load-balancers: ""

#     # Provides machine specific control plane configuration options.

#     # ControlPlane definition example.
#     controlPlane:
#          # Controller manager machine specific configuration options.
#          controllerManager:
#              disabled: false 

# sed -i 's/^\s*# allowSchedulingOnControlPlanes: true/    allowSchedulingOnControlPlanes: true/' _out/controlplane.yaml

# Change image in config file
vi  _out/controlplane.yaml
vi   _out/worker.yaml

install: 
  ...
  image: factory.talos.dev/installer/bf2113e1bea48d566f7d1e08eb780f832ccb56bbd7cf2f95769f7a04f9f2b184:v1.9.0

 #Save the file and fire up the control plane first:
# talosctl apply-config --insecure -n $CONTROL_PLANE_IP --file _out/controlplane.yaml

talosctl apply-config -f _out/controlplane.yaml -n $CONTROL_PLANE_IP -e $CONTROL_PLANE_IP --insecure
  export TALOSCONFIG=_out/talosconfig

mkdir -p $HOME/.kube
mkdir ~/.talos

sudo cp _out/talosconfig $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo cp -i  $HOME/_out/talosconfig $HOME/.talos/config

# View the nodes in the cluster
kubectl get nodes -o wide

# 2. Run the Worker Nodes
# Single node skip this step
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
#Single node 
# Allow scheduling on control-plane node for single node clusters:
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

#4. Access Talos Powered Kubernetes Cluster
#Once the cluster is up, you can access and use it as desired to run the containerized workloads. But first, obtain the admin kubeconfig
talosctl --talosconfig _out/talosconfig kubeconfig .


curl -fsSL"https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" -o /usr/local/bin
chmod +x kubectl
kubectl version



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


### talos config

```shell
talosctl list /var/local-path-provisioner -n 192.168.11.61  --endpoints 192.168.11.61 --talosconfig _out/talosconfig 

talosctl edit machineconfig -n 192.168.11.61  --endpoints 192.168.11.61 --talosconfig _out/talosconfig

```


### Install kubevirt

Option： Delete kubevirt 
```shell
export RELEASE=$(curl https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
kubectl delete -f https://github.com/kubevirt/kubevirt/releases/download/$RELEASE/kubevirt-operator.yaml
kubectl delete -f https://github.com/kubevirt/kubevirt/releases/download/$RELEASE/kubevirt-cr.yaml
```


```shell

export RELEASE=$(curl https://storage.googleapis.com/kubevirt-prow/release/kubevirt/kubevirt/stable.txt)
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/$RELEASE/kubevirt-operator.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/$RELEASE/kubevirt-cr.yaml
kubectl -n kubevirt wait kv kubevirt --for condition=Available
kubectl get po -n kubevirt


```
### Install virtctl

```shell
VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
echo ${ARCH}
curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
chmod +x virtctl
sudo install virtctl /usr/local/bin
virtctl
```

### Deploy VM

```shell

kubectl apply -f https://kubevirt.io/labs/manifests/vm.yaml
kubectl get vms
virtctl start testvm
kubectl get vmi
kubectl get pods -n default | grep -i launcher

virtctl console --kubeconfig=$KUBECONFIG testvm
```


### Delete virt pod with Error or complated status 

```shell
 for virtpod in $(k get po -A | grep -e 'Error' -e 'Completed' | awk '{print $2}'); do k delete pod $virtpod -n kubevirt ;done
```



### Upgrade talos
#### Upgrade talosctl

```bash
VER=$(curl -s https://api.github.com/repos/siderolabs/talos/releases/latest|grep tag_name|cut -d '"' -f 4|sed 's/v//')
echo $VER
curl -fsSL https://github.com/siderolabs/talos/releases/download/v${VER}/talosctl-linux-amd64 -o /usr/local/bin/talosctl
chmod +x /usr/local/bin/talosctl
/usr/local/bin/talosctl version
```
#### Upgrade talos node
```
NODE="192.168.11.50"
NEW_TALOS_IMAGE="factory.talos.dev/installer/bf2113e1bea48d566f7d1e08eb780f832ccb56bbd7cf2f95769f7a04f9f2b184:v1.9.0"
talosctl upgrade --nodes $NODE --image $NEW_TALOS_IMAGE --preserve=true
```


## Reference

  - [Install Kubernetes Cluster using Talos Container Linux](https://computingforgeeks.com/install-kubernetes-using-talos-container-linux/)
  - [How to enable workers on your control plane nodes](https://www.talos.dev/v1.6/talos-guides/howto/workers-on-controlplane/)
  - [Talos + Kubevirt Bare Metal & Nested Tenant Cluster](https://gist.github.com/usrbinkat/4dfe24590a56434139744fb7d1bc6ce9)
  - [Virtual Machine Orchestration in Kubernetes using KubeVirt](https://medium.com/@arbnair97/virtual-machine-orchestration-in-kubernetes-using-kubevirt-91bd0e81a5bd)
  - [Talos Linux (single-node) on Hetzner Robot servers](https://replicator.medium.com/talos-linux-single-node-on-hetzner-robot-servers-8ea8708c2f8f)

