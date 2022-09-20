---
title: "How to Reset Vyos to Default Settings"
date: 2020-09-04T22:48:34+08:00
lastmode: 2020-09-04T22:48:34+08:00
draft: false
tags: [ "vyos", "networking" ]
categories: ["networking"]
reward: true
mathjax: true
---
# How to reset VyOS to default settings

```bash
configure
load /opt/vyatta/etc/config.boot.default
commit
save
```



