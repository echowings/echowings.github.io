---
title: "How to Configure Debian 10 as a Docker Server"
date: 2019-07-26T16:58:29+08:00
lastmode: 2019-07-26T16:58:29+08:00
draft: false
tags: [ "docker", "debian","linux","pve" ]
categories: ["linux"]
reward: true
mathjax: true
---

# How to configure debian 10 as a docker server


After try to install docker in LXC with proxmox ve, there will be some issues to make docker stop working. At last, I gave up and brance pure linux as docker sever.

## Install debian 10 in promxox ve
Configure deiban 10 vm with 'qemu-agent' checked.


## After install deiban 10

  -  Change source list

```bash
mv /etc/apt/sources.list /etc/apt/sources.list.backup
wget https://mirrors.ustc.edu.cn/repogen/conf/debian-http-4-buster -O /etc/apt/sources.list
apt update && apt -y dist-upgrade

apt install -y openssh-server git vim cloud-init apt-transport-https
sed -i 's/http/https/g' /etc/apt/sources.list
apt update && apt -y dist-upgrade

```
  - To fix perl error warning

### option 1:

```bash
export LANGUAGE=en_US.UTF-8
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
#locale-gen en_US.UTF-8
cat >> /etc/environment <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
EOF
```
### option 2:

``` bash
sudo echo "LC_ALL=en_US.UTF-8" >> /etc/environment
sudo echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
sudo echo "LANG=en_US.UTF-8" > /etc/locale.conf
sudo locale-gen en_US.UTF-8
```

## vim setting

 - [How to Setup Vim](https://blog.stevedong.com/post/how-to-setup-vim/)



## cloud-init
 - [How to Setup Cloud Init in Proxmoxve](https://blog.stevedong.com/post/how-to-setup-cloud-init-in-proxmoxve/)

 
## Install docker on debian 10

 
### Step 1: Install Dependency packages
 
  Start the installation by ensuring that all the packages used by docker as dependencies are installed.
 
 ```bash
sudo apt-get -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
 
 ```
 
### Step 2: Add docker's offical GPG key:
Import Docker GPG key used for signing Docker packages:

```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
apt-key fingerprint 0EBFCD88
``` 

### Step 3: Add the docker repository to Debian 10
```bash
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```


### Step 4: Change docker repository to `mirrors.ustc.edu.cn`

```bash
sed -i 's/download.docker.com/mirrors.ustc.edu.cn\/docker-ce/g' /etc/apt/sources.list
```

### Step 5: Install docker and docker-compose on debian 10


```bash
apt update
apt -y install docker-ce docker-compose
```

### Step 6: Chang docker hub mirrors to `mirrors.ustc.edu.cn`

```bash
cat << EOF > /etc/default/docker
DOCKER_OPTS="--registry-mirror=https://docker.mirrors.ustc.edu.cn/"
EOF
systemctl restart docker
```




## Change Some settings

### Enable password authentication

```bash
sudo sed -i 's/PasswordAuthentication\ no/PasswordAuthentication\ yes/g' /etc/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd
```

### Permit root can loing with password

```bash
sudo sed -i 's/#PermitRootLogin\ prohibit-password/PermitRootLogin\ yes/g' /etc/ssh/sshd_config
sudo systemctl daemon-reload
sudo systemctl restart sshd
```

### Change machine-id

```bash
rm /var/lib/dbus/machine-id
truncate -s 0  /etc/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id
```

---THE END---



## Swell

1. [Cannot set LC_CTYPE to default locale: No such file or directory](https://askubuntu.com/questions/599808/cannot-set-lc-ctype-to-default-locale-no-such-file-or-directory)