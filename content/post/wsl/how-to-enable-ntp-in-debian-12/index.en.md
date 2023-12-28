---
title: "How to Enable Ntp in Debian 12"
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
sudo sed -i '/ConditionVirtualization/c\ConditionVirtualization=' /etc/systemd/sys
tem/sysinit.target.wants/systemd-timesyncd.service
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