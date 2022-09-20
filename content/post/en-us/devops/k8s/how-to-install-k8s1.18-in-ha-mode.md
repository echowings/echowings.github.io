---
title: "How to Install K8S"
date: 2021-04-08T18:03:14+08:00
lastmode: 2021-04-08T18:03:14+08:00
draft: false
tags: [ "k8s", "devops","docker" ]
categories: ["devops"]
reward: true
mathjax: true
---
# How to Install K8S

[TOC]


## Prerequest

  1. 下载所需的yaml文件 
  
   下载地址:[https://github.com/luckylucky421/kubernetes1.17.3/tree/master](https://github.com/luckylucky421/kubernetes1.17.3/tree/master)
  2. 初始化k8s集群需要的镜像

   下载链接: [https://pan.baidu.com/s/1k1heJy8lLnDk2JEFyRyJdA](https://pan.baidu.com/s/1k1heJy8lLnDk2JEFyRyJdA)
   提取码: `udkj`

## Prepare 5 centos 

  - Set timezone on centos

  ```bash
  timedatectl set-timezone Asia/Chongqing
  ```
  
  - Set hostname
  
  ```bash
  hostnamectl set-hostname xak8smaster01
  ```
  - Modifiy yum sourcelist

```bash
    
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
curl -fo /etc/yum.repos.d/CentOS-Base.repo -L http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache fast
  
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

yum clean all

yum makecache fast

yum -y update

yum -y install yum-utils device-mapper-persistent-data  lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum clean all

yum makecache fast

yum -y install wget net-tools nfs-utils lrzsz gcc gcc-c++ make cmake libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel wget vim ncurses-devel autoconf automake zlib-devel  python-devel epel-release openssh-server socat  ipvsadm conntrack ntpdate

systemctl stop firewalld  && systemctl  disable  firewalld
yum install iptables-services -y
 
service iptables stop   && systemctl disable iptables
 
ntpdate cn.pool.ntp.org
 
#crontab -e
#* */1 * * * /usr/sbin/ntpdate   cn.pool.ntp.org

crontab -l | { cat; echo "* */1 * * * /usr/sbin/ntpdate   cn.pool.ntp.org"; } | crontab -

service crond restart

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
reboot -r
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

cat << EOF >> /etc/hosts
10.32.0.91 xak8smaster01
10.32.0.92 xak8smaster02
10.32.0.93 xak8smaster03
10.32.0.94 xak8smaster04
10.32.0.95 xak8snode01
10.32.0.96 xak8snode02
EOF

ssh-keygen -t rsa

sed -i 's/#PermitRootLogin\ yes/PermitRootLogin\ yes/g' /etc/ssh/sshd_config

sed -i 's/PasswordAuthentication\ no/PasswordAuthentication\ yes/g' /etc/ssh/sshd_config
cat /etc/ssh/sshd_config | grep PermitRootLogin
cat /etc/ssh/sshd_config | grep PasswordAuthentication
systemctl daemon-reload
systemctl restart sshd


ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster03
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster04
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode02

ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster03
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster04
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode02


ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster04
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode02


ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster03
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode02

cat << EOF >> 1.sh
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster03
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster04
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode02
EOF
sh 1.sh

cat << EOF >> 1.sh
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster01
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster02
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster03
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8smaster04
ssh-copy-id -i ~/.ssh/id_rsa.pub root@xak8snode01
EOF
sh 1.sh


yum list docker-ce --showduplicates |sort -r



cat > /etc/docker/daemon.json <<EOF
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "log-driver": "json-file",
 "log-opts": {
   "max-size": "100m"
  },
 "storage-driver": "overlay2",
 "storage-opts": [
   "overlay2.override_kernel_check=true"
  ]
}
EOF


systemctl daemon-reload && systemctl restart docker


echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
echo 1 >/proc/sys/net/bridge/bridge-nf-call-ip6tables
echo """
vm.swappiness = 0
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
""" > /etc/sysctl.conf
sysctl -p



cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_fo ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack"
for kernel_module in \${ipvs_modules}; do
 /sbin/modinfo -F filename \${kernel_module} > /dev/null 2>&1
 if [ $? -eq 0 ]; then
 /sbin/modprobe \${kernel_module}
 fi
done
EOF


yum install -y conntrack-tools ipvsadm ipset conntrack libseccomp
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep ip_vs




cat > /etc/sysctl.d/k8s.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp.keepaliv.probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp.max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp.max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.top_timestamps = 0
net.core.somaxconn = 16384
EOF
 
sysctl --system







docker pull k8s.gcr.io/kube-apiserver:v1.18.2
docker pull k8s.gcr.io/kube-controller-manager:v1.18.2
docker pull k8s.gcr.io/kube-scheduler:v1.18.2
docker pull k8s.gcr.io/kube-proxy:v1.18.2

```



