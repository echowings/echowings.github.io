---
title: "How to Enable NTP in Debian 12"
date: 2023-12-28T19:03:56+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=242a325a
categories:
  - linux
tags:
  - wsl
  - linux
  - debian
  - time
  - timesyncd
slug:
  - debian
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


## Install `system-timesyncd`
1. Uninstall `ntpd`
```shell
sudo apt purget ntp
```

2. Install `timesyncd`
```shell
sudo apt install -y systemd-timesyncd
```

3. Start `timesyncd` service
```shell
sudo systemctl start systemd-timesyncd
```

4. enable service auto started
```shell
sudo systemctl enable systemd-timesyncd
```

5. Check the status:
```shell
sudo systemctl status system-timesyncd
```

6. Disply the current time and date.
```shell
timedatectl
```
## Fix can't start on wsl

1. Modify config

```shell
#sudo sed -i '/ConditionVirtualization/c\ConditionVirtualization=' /etc/systemd/system/sysinit.target.wants/systemd-timesyncd.service
# Fix time sync issues in WSL
# https://github.com/microsoft/WSL/issues/8204#issuecomment-1338334154
sudo mkdir -p /etc/systemd/system/systemd-timesyncd.service.d
sudo tee /etc/systemd/system/systemd-timesyncd.service.d/override.conf > /dev/null <<'EOF'
[Unit]
ConditionVirtualization=
EOF
sudo systemctl start systemd-timesyncd
sudo systemctl enable systemd-timesyncd
```

2. Reboot service
```shell
sudo systemctl daemon-reload   
sudo systemctl restart systemd-timesyncd
```
3. Check `systemd-timesyncd` service status

```shell
sudo systemctl status systemd-timesyncd
```

## with script 

```shell
systemd_timesyncd_exist=$(sudo systemctl is-active systemd-timesyncd | grep active | wc -l)
echo $systemd_timesyncd_exist
if [ $systemd_timesyncd_exist -ne 1 ]
then
    sudo apt update -y
    sudo apt purget ntp -y
    sudo apt install -y systemd-timesyncd
    sudo systemctl start systemd-timesyncd
    sudo systemctl enable systemd-timesyncd
fi
sudo mkdir -p /etc/systemd/system/systemd-timesyncd.service.d
echo 'hello'
sudo tee /etc/systemd/system/systemd-timesyncd.service.d/override.conf > /dev/null <<'EOF'
[Unit]
ConditionVirtualization=
EOF

sudo systemctl daemon-reload   
sudo systemctl restart systemd-timesyncd

```
