---
title: "Proxmox Ve How to Chang Network of Hosting Without Reboot Proxmox Ve"
date: 2019-04-27T11:36:16+08:00
draft: false
tags: ["pve", "networking"]
categories: ["virtualization"]
reward: false
---

# How to change Network settings of proxmox-ve host without rebooting.

Eache time when  I want to change network settings like change netowrks and I must reboot to apply it. It is really trouble me.


## Solution

Proxmox VE is based on debian. After check interface of network with the path `/etc/network/` 

I found that each time when i change network settings. There will be a named `interfaces.new` created and it content new network settings wait for apply when i reboot. So I try to backup the old networking interfaces settings and replace it with the new file.

### Manual change
  -  Backup interfaces

``` shell
cp /etc/network/interfaces /etc/network/interfaces.old

```
  -  Copy interface.new to interface

``` shell
cp /etc/network/interfaces.new /etc/network/interfaces

```

  -  Reboot networking service

```shell

systemctl restart networking

```
### Scirpt change

```bash
mv interfaces interfaces.backup && mv interfaces.new interfaces && systemctl restart networking.service
```


