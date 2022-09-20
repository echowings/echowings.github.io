---
title: "Macos Update Time via Ntpserver"
date: 2018-12-26T09:26:04+08:00
draft: false
tags: ['macos', 'ntp']
categories: ['macos']
author: "steve Dong"
reward: true
---

# How to update time via ntpserver

Today, My MBP is out of power. I cannot surfing internet. and the brower tell me my time was wrong. Then I checked my time in my MBB, I'm back in time jan 1, 2018. Wow, I got extra whole year time!

# Majove update time

``` bash
sudo sntp -sS time.asia.apple.com


```

When you're getting this error

``` bash

kod_init_kod_db(): Cannot open KoD db file /var/db/ntp-kod: No such file or directory

```

You can solved the problems with:

``` bash
sudo touch /var/db/ntp-kod
sudo chmod 666 /var/db/ntp-kod

```

## Disable No subscription warning

``` bash
sed -i.bak "s/data.status !== 'Active'/false/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

## Reference

* [How can I tell if my Mac is keeping the clock updated properly?](https://apple.stackexchange.com/questions/117864/how-can-i-tell-if-my-mac-is-keeping-the-clock-updated-properly?noredirect=1&lq=1)