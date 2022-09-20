---
title: "Install Proxmox VE  6.x"
date: 2019-07-17T16:07:55+08:00
lastmode: 2019-07-17T16:07:55+08:00
draft: false
tags: [ "pve" ]
categories: ["virtualization"]
reward: true
mathjax: true
---

# Install Proxmox VE  6.x 

After installed Proxmox VE 6.x, What we gone  do ?

## Download Proxmox VE 6.x iSO file in mirrors.ustc.edu.cn

Download iso images from here:


[http://mirrors.ustc.edu.cn/proxmox/iso/](http://mirrors.ustc.edu.cn/proxmox/iso/)


## Fixe Perl error waring

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

## Disable subscription Warning



  - Disable subscription warning via script

  below 6.3

 ```bash
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

  - Manually disable subscription warning

```bash
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
nano proxmoxlib.js

# change the value as show below
if (data.status !== 'Active') {
#to
if (false) {

#restart pveproxy.service
systemctl restart pveproxy.service
```

 above 6.3
 
 ```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
 ```
 ### manual setup
 Manual Steps
Here are alternative step by step instructions so you can understand what the above command is doing:

1. Change to working directory

```bash
cd /usr/share/javascript/proxmox-widget-toolkit
```

2. Make a backup

```bash
cp proxmoxlib.js proxmoxlib.js.bak
```
3. Edit the file

```bash
nano proxmoxlib.js
```
4. Locate the following code
(Use ctrl+w in nano and search for “No valid subscription”)

```js
Ext.Msg.show({
  title: gettext('No valid subscription'),
```

5. Replace “Ext.Msg.show” with “void”

```js
void({ //Ext.Msg.show({
  title: gettext('No valid subscription'),
```
6. Restart the Proxmox web service (also be sure to clear your browser cache, depending on the browser you may need to open a new tab or restart the browser)

```bash
systemctl restart pveproxy.service
```
Additional Notes
You can quickly check if the change has been made:

```bash
grep -n -B 1 'No valid sub' proxmoxlib.js
```

You have three options to revert the changes:

  - Manually edit  proxmoxlib.js to to undo the changes you made
  - Restore the backup file you created from the proxmox-widget-toolkit directory

```bash
mv proxmoxlib.js.bak proxmoxlib.js
```

  - Reinstall the proxmox-widget-toolkit package from the repository

```bash
apt-get install --reinstall proxmox-widget-toolkit
```

## Chang  apt sourcelist to mirrors.ustc.edu.cn

``` bash
#Backup sources.list
mv /etc/apt/sources.list /etc/apt/sources.list.backup


# Change debian 10 sourcelist to mirrors.ustc.edu.cn
# https://mirrors.ustc.edu.cn/repogen/

cat << EOF > /etc/apt/sources.list
deb https://mirrors.ustc.edu.cn/debian/ buster main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ buster main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free

deb https://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free
EOF

# Add pve-no-subscription with mirrors.ustc.edu.cn
#change pve 5.x update source list
cat << EOF >> /etc/apt/sources.list
deb https://mirrors.ustc.edu.cn/proxmox/debian/pve/ buster pve-no-subscription
EOF

##change ceph source list to utsc mirrors
cat << EOF >>  /etc/apt/sources.list
deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-nautilus/ buster main
EOF

# Add Key
wget https://mirrors.ustc.edu.cn/proxmox/debian/proxmox-ve-release-6.x.gpg -O /etc/apt/trusted.gpg.d/proxmox-ve-release-6.x.gpg


# Delete `pve-enterprise.list`
rm -rf /etc/apt/sources.list.d/pve-enterprise.list

# Upgrade pve
apt update && apt -y dist-upgrade
pveupdate && pveupgrade -y


# update grub settings
update-grub


```

## Install openvswitch

``` bash
apt install -y openvswitch-switch
```






